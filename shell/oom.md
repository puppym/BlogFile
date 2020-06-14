When（什么时候出现）：

[linux](http://lib.csdn.net/base/linux)下允许程序申请比系统可用内存更多的内存，这个特性叫Overcommit。这样做是出于优化系统考虑，因为不是所有的程序申请了内存就立刻使用的，当你使用的时候说不定系统已经回收了一些资源了。不幸的是，当你用到这个Overcommit给你的内存的时候，系统还没有资源的话，OOM killer就跳出来了。

参数/proc/sys/vm/overcommit_memory可以控制进程对内存过量使用的应对策略
1.当overcommit_memory=0 允许进程轻微过量使用内存，但对于大量过载请求则不允许，也就是当内存消耗过大就是触发OOM killer。
2.当overcommit_memory=1 永远允许进程overcommit，不会触发OOM killer。
3.当overcommit_memory=2 永远禁止overcommit，不会触发OOM killer。

 

How（系统会怎么样）：

当然，如果触发了OOM机制，系统会杀掉某些进程，那么什么进程会被处理掉呢？kernel提供给用户态的/proc下的一些参数：
1./proc/[pid]/oom_adj，该pid进程被oom killer杀掉的权重，介于 [-17,15]（具体具体权重的范围需要查看内核确认）之间，越高的权重，意味着更可能被oom killer选中，-17表示禁止被kill掉。

通过2个步骤可以确认，具体权重的范围：

①uname -a查看Linux内核版本

②进入/usr/src/kernels/内核版本/include/linux/oom.h确认具体的权重范围

![img](http://img.blog.csdn.net/20160725160127732?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
2./proc/[pid]/oom_score，当前该pid进程的被kill的分数，越高的分数意味着越可能被kill，这个数值是根据oom_adj运算（2ⁿ，n就是oom_adj的值）后的结果。

oom_adj，oom_score是oom_killer的主要参考值

 

 

So（我们能做什么）：

1.保护我们重要的进程，避免被处理掉

实例：

ps -ef|grep GameServer（获得重要进程的PID）

echo -17 > /proc/PID/oom_score_adj（输入-17，禁止被OOM机制处理）

2.关闭OOM机制（不推荐，如果不启动OOM机制，内存使用过大，会让系统产生很多异常数据）

echo "vm.panic_on_oom=1" >> /etc/sysctl.conf

systcl -p



SmartCheck: Static Analysis of Ethereum Smart Contracts