### 显示linux的某一行

1. tail和head组合使用

从第1000行开始，显示2000行。即显示1000~2999行

cat input_file | tail -n +1000 | head -n 2000

2. 

显示 1000行到3000行

cat input_file | head -n 3000 | tail -n +1001



tail -n 1000：显示最后1000行

tail -n +1000：从1000行开始显示，显示1000行以后的

head -n 1000：显示前面1000行

3.  sed -n '5,10p' input_file这样你就可以只查看文件的第5行到第10行。

4. 用awk处理

   awk 'NR==2, NR==11{print}'  input_file

   或者

   awk 'NR>2 && NR<11 {print $0}'  input_file

https://blog.csdn.net/wu8439512/article/details/78642892



### linux查看系统用了哪些信号量用什么命令？

ipcs用法 
ipcs -a  是默认的输出信息 打印出当前系统中所有的进程间通信方式的信息
ipcs -m  打印出使用共享内存进行进程间通信的信息
ipcs -q   打印出使用消息队列进行进程间通信的信息
ipcs -s  打印出使用信号进行进程间通信的信息

ipcs -t   输出信息的详细变化时间

ipcs -u  输出当前系统下ipc各种方式的状态信息(共享内存，消息队列，信号)

**

**ipcrm 命令** 
移除一个消息对象。或者共享内存段，或者一个信号集，同时会将与ipc对象相关链的数据也一起移除。当然，只有超级管理员，或者ipc对象的创建者才有这项权利

ipcrm -M shmkey  移除用shmkey创建的共享内存段
ipcrm -m shmid    移除用shmid标识的共享内存段
ipcrm -Q msgkey  移除用msqkey创建的消息队列
ipcrm -q msqid  移除用msqid标识的消息队列
ipcrm -S semkey  移除用semkey创建的信号
ipcrm -s semid  移除用semid标识的信号

### 讲讲进程间的通信机制？你熟悉其中的哪些方式？

管道

消息队列

共享内存

信号量



##  僵尸进程和孤儿进程

**孤儿进程：一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。**

　　**僵尸进程：一个进程使用fork创建子进程，如果子进程退出，而父进程并没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵死进程。**

https://www.cnblogs.com/anker/p/3271773.html


## free里面空闲内存是实际的空闲内存？

```
[root@VM_16_17_centos bin]# free 



              total        used        free      shared  buff/cache   available



Mem:        1882892      785272      280428       40496      817192      852060
Swap:             0           0           0
```

先说明一些基本概念
**第一列**
`Mem` 内存的使用信息
`Swap` 交换空间的使用信息
**第一行**
`total` 系统总的可用物理内存大小
`used` 已被使用的物理内存大小
`free` 还有多少物理内存可用
`shared` 被共享使用的物理内存大小
`buff/cache` 被 buffer 和 cache 使用的物理内存大小
`available` 还可以被 ***应用程序\*** 使用的物理内存大小

其中有两个概念需要注意

### free 与 available 的区别

`free` 是真正尚未被使用的物理内存数量。
`available` 是应用程序认为可用内存数量，`available = free + buffer + cache` (注：只是大概的计算方法)

https://blog.csdn.net/gpcsy/article/details/84951675	

## 加密算法

哈希

对称加密 AES DES 3DES

非对称加密 RSA ECC  ECDSA  EdDSA

https://juejin.cn/post/6844903638117122056

### awk 和 sed 

awk语言的最基本功能是在文件或者字符串中基于指定规则浏览和抽取信息，把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。

awk命令形式:
awk [-F|-f|-v]  'commands' input-file(s)
 [-F|-f|-v] -F指定分隔符，-f调用脚本，-v定义变量 var=value
'commands  '      可以是引用代码块 'BEGIN{}  {command1; command2} END{}'
BEGIN   初始化代码块，在对每一行进行处理之前，初始化代码，主要是引用全局变量，命令代码块可包含一条或多条命令
END 结尾代码块，在对每一行进行处理之后再执行的代码块，主要是进行最终计算或输出结尾摘要信息
如果只需要文本中某列信息,或某几列信息

sed是一种流编辑器，在文本处理中非常适用的工具，能够配合正则表达式使用。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有 改变，除非你使用重定向存储输出。

sed命令格式

sed [options] 'command' file(s) 
sed [options] -f scriptfile file(s)
原文链接：https://blog.csdn.net/yin1031468524/article/details/68105301



