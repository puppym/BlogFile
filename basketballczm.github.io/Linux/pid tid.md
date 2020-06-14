## pid和tid 总结 
进程pid： getpid() 
线程tid: pthread_self() 进程内唯一， 但是在不同进程则不唯一
线程pid： syscall(SYS_gettid) 系统内是唯一的，相当于进程的pid 

```c
#include <stdio.h>
#include <string.h>
#include <pthread.h> // for thread
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/syscall.h>

pthread_t tid[2];  // 创建两个线程

//所有线程共用一个进程的ID，每个线程有自己的线程ID
void *doSomeThing(void *arg) {
    unsigned long i = 0;
    pthread_t id = pthread_self();

    if (pthread_equal(id, tid[0])) {
        printf("\n First thread processing\n"); // 第一个线程正在执行
    } else {
        printf("\n Second thread processing\n");
    }

    //这里
    //父进程ID
    printf("child getpid = %u\n",  (unsigned)getpid());
    //线程ID，进程内唯一，但是在不同进程内可能会有相同值存在
    printf("child pthread_self= %u\n", (unsigned int)pthread_self());
    //线程pid，整个系统内是唯一 没有gettid的实现，只能通过系统调用syscall来进行获取，进程间的通讯要使用这个进程号
    //有时候我们可能需要知道线程的真实pid。比如进程P1要向另外一个进程P2中的某个线程发送信号时，既不能使用P2的pid，更不能使用线程的pthread id，而只能使用该线程的真实pid，称为tid。
    printf("child syscall(SYS_gettid)= %u\n", (unsigned int)syscall(SYS_gettid));

    sleep(200);
    return NULL;
}

int main(void) {
    int i = 0;
    int err;
    //两者相同
    printf("father getpid = %u\n",  (unsigned)getpid());
    printf("father syscall(SYS_gettid)= %u\n", (unsigned int)syscall(SYS_gettid));
    while (i < 2) {
        err = pthread_create(&(tid[i]), NULL, &doSomeThing, NULL);
        if (err != 0) printf("\n can't create thread:[%s]", strerror(err));
        else printf("\n Thread created successfully\n");

        i++;
    }
    sleep(300);
    return 0;
}
```
## 原文引用  
进程和线程都有自己的pid，这个ID就叫做pid，pid不是特指进程ID，线程ID也可以叫做PID。 
The four threads will have the same PID but only when viewed from above. What you (as a user) call a PID is not what the kernel (looking from below) calls a PID.

In the kernel, each thread has it's own ID, called a PID (although it would possibly make more sense to call this a TID, or thread ID) and they also have a TGID (thread group ID) which is the PID of the thread that started the whole process.

Simplistically, when a new process is created, it appears as a thread where both the PID and TGID are the same (new) number.

When a thread starts another thread, that started thread gets its own PID (so the scheduler can schedule it independently) but it inherits the TGID from the original thread.

That way, the kernel can happily schedule threads independent of what process they belong to, while processes (thread group IDs) are reported to you.

## 关系图
```c
                USER VIEW
 <-- PID 43 --> <----------------- PID 42 ----------------->
                     +---------+
                     | process |
                    _| pid=42  |_
                  _/ | tgid=42 | \_ (new thread) _
       _ (fork) _/   +---------+                  \
      /                                        +---------+
+---------+                                    | process |
| process |                                    | pid=44  |
| pid=43  |                                    | tgid=42 |
| tgid=43 |                                    +---------+
+---------+
 <-- PID 43 --> <--------- PID 42 --------> <--- PID 44 --->
                     KERNEL VIEW
```

## 高效查看命令 
1. top 
2. ps -elf | grep java    e显示全部进程，L显示线程，f全格式输出 
3. pstree -p pid 不加pid显示所有 
4. top -Hp pid   实时 
