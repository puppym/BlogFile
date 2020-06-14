sudo chmod 4755 backdoor //修改文件使用权限
4 任何人执行该程序时其权限是等于owner用户的权限，因此普通用户也能通过设置获取root权限。
用户ID和组ID
root:x:0:0
seteid()  //修改有效ID
setuid(0) //通过一定权限需要是root权限，将当前用户换为root用户，修改用户ID
/usr/bin/bash 获取的权限使根据real id, /usr/bin/sh 获取的权限使根据effective , effective id能够使用setuid进行更改。

经过以上的讨论，我们再来看一下，如果希望远程获得一个root shell，需要怎么做呢？
setuid(0) -> 需要root权限 -> backdoor owner root(暂时root权限) -> chmod 4755 backdoor(暂时root权限) 
后门->rootkit   长期获取root权限，但是暂时获取root权限时简单

effective id能够保存高权限的ID

real ID / effective ID

/bin/ls -> mysecls
攻击者修改readdir库函数 getlents
# rootkit1 
Linux的用户管理机制中，能否访问文件的特权是基于用户id和组id的。当用户需要改变权限的时候，就需要更换用户ID或者组ID。为了实现这种机制，引入了真实UID（real UID）、有效UID（effective UID）以及 保存的UID（saved set-user-ID）的概念。

1. 实际用户ID和实际组ID标示我们实际上是谁；这两个字段是在登录时读取口令文件中的登录项。一般情况下，在一个会话期间，实际用户和实际组用户不会改变；但超级用户的进程可能改变它们。
2. 有效用户ID，有效组ID以及附加组ID决定了文件访问权限；
3. 保存的设置用户ID在执行程序时包含了有效用户ID的副本。

**setuid和seteuid** 
1. setuid(uid)首先请求内核将本进程的[真实uid],[有效uid]和[被保存的uid]都设置成函数指定的uid, 若权限不够则请求只将effective uid设置成uid, 再不行则调用失败.
2. seteuid(uid)仅请求内核将本进程的[有效uid]设置成函数指定的uid.

**setuid**
1. 当用户具有超级用户权限的时候,setuid 函数设置的id对三者都起效.[规则一]
2. 否则,仅当该id为real user ID或者saved user ID时,该id对effective user ID起效.[规则二]
3. 否则,setuid函数调用失败.


两个例子 
```c
#include <stdio.h>
#include <stdlib.h>

int main(){
        printf("Real id is %d, Effective id is %d.\n",getuid(),geteuid());
        seteuid(1000);
        printf("Real id is %d, Effective id is %d.\n",getuid(),geteuid());
        setuid(0);
        printf("Real id is %d, Effective id is %d.\n",getuid(),geteuid());
}


czm@ubuntu:~/work$ ./1
Real id is 1000, Effective id is 0.
Real id is 1000, Effective id is 1000.
Real id is 1000, Effective id is 0.
```


```c
#include <stdio.h>
#include <stdlib.h>

int main(){
        printf("Real id is %d, Effective id is %d.\n",getuid(),geteuid());
        setuid(1000);
        printf("Real id is %d, Effective id is %d.\n",getuid(),geteuid());
        setuid(0);
        printf("Real id is %d, Effective id is %d.\n",getuid(),geteuid());
}

root@ubuntu:/home/czm/work# ./1
Real id is 0, Effective id is 0.
Real id is 1000, Effective id is 1000.
Real id is 1000, Effective id is 1000.

```
结果：

```c
#include <stdio.h>
#include <stdlib.h>

int main(){
        printf("Real id is %d, Effective id is %d.\n",getuid(),geteuid());
        seteuid(1001);
        printf("Real id is %d, Effective id is %d.\n",getuid(),geteuid());
        setuid(1000);
        printf("Real id is %d, Effective id is %d.\n",getuid(),geteuid());
        setuid(0);
        printf("Real id is %d, Effective id is %d.\n",getuid(),geteuid());


}

-rwsr-xr-x  1 root root   8480 Apr 25 05:42 1

czm@ubuntu:~/work$ ./1
Real id is 1000, Effective id is 0.
Real id is 1000, Effective id is 1001. //通过seteuid改变值之后，能够通过setuid返回
Real id is 1000, Effective id is 1000.
Real id is 1000, Effective id is 0.

```


# rootkit2 
首先找IDT中断向量表的表头，由于中断使用比较频繁，一般会存在一个寄存器中，通过中断号和中断向量表找到系统调用中断处理程序的函数地址(这个函数会做一些环境保存和恢复的工作，以及通过系统函数表和eax来查找要调用系统函数的地址)，通过查找系统调用函数的call指令，call的函数地址可以找到sys_call_table的地址(SCT表一般都有写保护，所以，直接去修改会把醋，首先要对它取消写保护，可以通过设置cr0寄存器的WP位为0，进制CPU上的写保护)，根据系统调用表可以找到所要调用的系统函数，并且进行调用。
