## 消息队列内核锁操作分析
### int msgrcv(int msqid, struct msgbuf \*msgp, int msgsz, long msgtyp, int msgflg);
msgrcv()解除阻塞的条件有三个：
1. 消息队列中有了满足条件的消息；
2. msqid代表的消息队列被删除；
3. 调用msgrcv（）的进程被信号中断；
调用返回：成功返回读出消息的实际字节数，否则返回 -1。

### int msgsnd(int msqid, struct msgbuf \*msgp, int msgsz, int msgflg);
对发送消息来说，有意义的msgflg标志为IPC_NOWAIT，指明在消息队列没有足够空间容纳要发送的消息时，msgsnd是否等待。造成msgsnd()等待的条件有两种：
1. 当前消息的大小与当前消息队列中的字节数之和超过了消息队列的总容量。

msgsnd()解除阻塞的条件有3个
1. 消息队列中有容纳该消息的空间。
2. msqid代表的消息队列被删除。
3. 调用msgsnd()的进程被信号中断。

### 消息队列内核实现分析
查看内核版本: apt - cache search linux - source
内核源代码对应的目录为 / usr / src
使用内核源码查看的网站： https://elixir.bootlin.com/linux/v4.15.17/source

```c
// 先看msgsnd()函数，它通过系统调用接口界面，进入内核执行，代码如下：
SYSCALL_DEFINE4(msgsnd, int, msqid, struct msgbuf __user *, msgp, size_t, msgsz,
                int, msgflg) {
    long mtype;

    if (get_user(mtype, &msgp->mtype))
        return -EFAULT;
    return do_msgsnd(msqid, mtype, msgp->mtext, msgsz, msgflg);
}
// 接下来看do_msgsnd()部分的代码，如下：
long do_msgsnd(int msqid, long mtype, void __user *mtext,
               size_t msgsz, int msgflg) {
    struct msg_queue *msq;
    struct msg_msg *msg;
    int err;
    struct ipc_namespace *ns;

    ns = current->nsproxy->ipc_ns;

    if (msgsz > ns->msg_ctlmax || (long) msgsz < 0 || msqid < 0)
        return -EINVAL;
    if (mtype < 1)
        return -EINVAL;

    //填入消息内容
    msg = load_msg(mtext, msgsz);
    if (IS_ERR(msg))
        return PTR_ERR(msg);

    //填入msg_type
    msg->m_type = mtype;
    msg->m_ts = msgsz;

    //消息队列上锁检查
    msq = msg_lock_check(ns, msqid);
    if (IS_ERR(msq)) {
        err = PTR_ERR(msq);
        goto out_free;
    }

    for (;;) {
        struct msg_sender s;

        err = -EACCES;
        if (ipcperms(&msq->q_perm, S_IWUGO))
            goto out_unlock_free;

        err = security_msg_queue_msgsnd(msq, msg, msgflg);
        if (err)
            goto out_unlock_free;

        if (msgsz + msq->q_cbytes <= msq->q_qbytes &&
                1 + msq->q_qnum <= msq->q_qbytes) {
            break;
        }

        /* queue full, wait: */
        if (msgflg & IPC_NOWAIT) {
            err = -EAGAIN;
            goto out_unlock_free;
        }
        ss_add(msq, &s);
        ipc_rcu_getref(msq);
        //解锁消息队列
        msg_unlock(msq);
        schedule();

        ipc_lock_by_ptr(&msq->q_perm);
        ipc_rcu_putref(msq);
        if (msq->q_perm.deleted) {
            err = -EIDRM;
            goto out_unlock_free;
        }
        ss_del(&s);

        if (signal_pending(current)) {
            err = -ERESTARTNOHAND;
            goto out_unlock_free;
        }
    }

    msq->q_lspid = task_tgid_vnr(current);
    msq->q_stime = get_seconds();

    if (!pipelined_send(msq, msg)) {
        /* noone is waiting for this message, enqueue it */
        list_add_tail(&msg->m_list, &msq->q_messages);
        msq->q_cbytes += msgsz;
        msq->q_qnum++;
        atomic_add(msgsz, &ns->msg_bytes);
        atomic_inc(&ns->msg_hdrs);
    }

    err = 0;
    msg = NULL;

out_unlock_free:
    msg_unlock(msq);
out_free:
    if (msg != NULL)
        free_msg(msg);
    return err;
}
// 在这段代码中，请注意临近入口位置的这个函数msg_lock_check()，我们跟进，看一下这个lock是如何check
// 的，代码如下：
static inline struct msg_queue *msg_lock_check(struct ipc_namespace *ns,
        int id) {
    struct kern_ipc_perm *ipcp = ipc_lock_check(&msg_ids(ns), id);

    if (IS_ERR(ipcp))
        return (struct msg_queue *)ipcp;

    return container_of(ipcp, struct msg_queue, q_perm);
}
// ipc_lock_check()是一个能够check所有IPC object同步信息的函数，它的定义如下：
struct kern_ipc_perm *ipc_lock_check(struct ipc_ids *ids, int id) {
    struct kern_ipc_perm *out;

    out = ipc_lock(ids, id);
    if (IS_ERR(out))
        return out;

    if (ipc_checkid(out, id)) {
        ipc_unlock(out);
        return ERR_PTR(-EIDRM);
    }

    return out;
}
// 这里的ipc_lock()是至关重要的地方！通过这个函数的注释，也能明白它的作用了：
/**
 * ipc_lock - Lock an ipc structure without rw_mutex held
 * @ids: IPC identifier set
 * @id: ipc id to look for
 *
 * Look for an id in the ipc ids idr and lock the associated ipc object.
 *
 * The ipc object is locked on exit.
 */

struct kern_ipc_perm *ipc_lock(struct ipc_ids *ids, int id) {
    struct kern_ipc_perm *out;
    int lid = ipcid_to_idx(id);

    rcu_read_lock();
    out = idr_find(&ids->ipcs_idr, lid);
    if (out == NULL) {
        rcu_read_unlock();
        return ERR_PTR(-EINVAL);
    }

    spin_lock(&out->lock);

    /* ipc_rmid() may have already freed the ID while ipc_lock
     * was spinning: here verify that the structure is still valid
     */
    if (out->deleted) {
        spin_unlock(&out->lock);
        rcu_read_unlock();
        return ERR_PTR(-EINVAL);
    }

    return out;
}

```

## 参考文档
* https://bbs.csdn.net/topics/300199658 

* Linux程序设计.pdf 432页(与管道例子不同的是，不再需要由进程自己来提供同步机制了，这是消息队列相对于管道的一大优势)