# linux进程通讯
##进程通信有如下一些目的：
1. 数据传输：一个进程需要将它的数据发送给另一个进程，发送的数据量在一个字节到几M字节之间
2. 共享数据：多个进程想要操作共享数据，一个进程对共享数据的修改，别的进程应该立刻看到。
3. 通知事件：一个进程需要向另一个或一组进程发送消息，通知它（它们）发生了某种事件（如进程终止时要通知父进程）。
4. 资源共享：多个进程之间共享同样的资源。为了作到这一点，需要内核提供锁和同步机制。
5. 进程控制：有些进程希望完全控制另一个进程的执行（如Debug进程），此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变。

Linux 进程间通信（IPC）以下以几部分发展而来：
早期UNIX进程间通信、基于SystemV进程间通信、基于Socket进程间通信和POSIX进程间通信。
UNIX进程间通信方式包括：管道、FIFO、信号。
System V进程间通信方式包括：System V消息队列、SystemV信号灯、System V共享内存、
POSIX进程间通信包括：posix消息队列、posix信号灯、posix共享内存。

## linux进程通讯的主要方式

1. 管道(pipe)和有名管道(FIFO)：ps | grep vsftpd
有名管道和无名管道都是半双工的通讯，但是无名管道只允许具有亲缘关系的父子进程进行通讯，有名管道允许不具有亲缘关系的进程之间进行通讯。

2. 信号(signal)：QT 编程中的信号和槽的机制。
信号是一种比较复杂的通讯方式，用于通知接收进程某个事件已经发生。

3. 消息队列：windows编程中的消息环，通过消息环接收所有消息，然后通过校验不同的消息进行不同的动作。
消息队列是消息的链表, 存放在内核中并由消息队列标识符标识.**消息队列克服了信号传递信息少, 管道只能承载无格式字节流以及缓冲区大小受限等特点**.消息队列是UNIX下不同进程之间可实现共享资源的一种机制, **UNIX允许不同进程将格式化的数据流以消息队列形式发送给任意进程.**对消息队列具有操作权限的进程都可以使用msget完成对消息队列的操作控制.通过使用消息类型, 进程可以按任何顺序读信息, 或为消息安排优先级顺序.

4. 共享内存：进程共享内存从而进行数据的共享。
**共享内存就是映射一段能被其他进程所访问的内存, 这段共享内存由一个进程创建, 但多个进程都可以访问.共享内存是最快的IPC(进程间通信)方式, 它是针对其它进程间通信方式运行效率低而专门设计的.**它往往与其他通信机制, 如信号量, 配合使用, 来实现进程间的同步与通信.

5. 信号量：进程同步的主要方式，信号量限定了每次执行临界区进程的个数。
信号量是一个计数器，可以用来控制多个线程对于共享资源的访问，它不是用于交换大批数据, 而用于多线程之间的同步.它常作为一种锁机制, 防止某进程在访问资源时其它进程也访问该资源.因此, 主要作为进程间以及同一个进程内不同线程之间的同步手段.

6. 套接字(socket)：每个进程可以通过socket进行通讯。
socket，即套接字是一种通信机制，凭借这种机制，客户 / 服务器（即要进行通信的进程）系统的开发工作既可以在本地单机上进行，也可以跨网络进行。也就是说**它可以让不在同一台计算机但通过网络连接计算机上的进程进行通信**也因为这样，套接字明确地将客户端和服务器区分开来。

**管道读写**
```c
#include<unistd.h>
#include<sys/types.h>
#include<sys/wait.h>
#include<memory.h>
#include<errno.h>
#include<stdio.h>
#include<stdlib.h>
int main() {
    int pipe_fd[2];
    pid_t pid;
    char buf_r[100];
    char* p_wbuf;
    int r_num;
    memset(buf_r, 0, sizeof(buf_r));
    if (pipe(pipe_fd) < 0) {
        printf("pipe create error\n");
        return -1;
    }
    if ((pid = fork()) == 0) {
        //child
        printf("\n");
        close(pipe_fd[1]);
        sleep(2);
        if ((r_num = read(pipe_fd[0], buf_r, 100)) > 0) {
            printf("%d numbers read from be pipe is %s\n", r_num, buf_r);
        }
        close(pipe_fd[0]);
        exit(0);
    } else if (pid > 0) {
        //father
        close(pipe_fd[0]);
        if (write(pipe_fd[1], "Hello", 5) != -1)
            printf("parent write success!\n");
        if (write(pipe_fd[1], " Pipe", 5) != -1)
            printf("parent wirte2 succes!\n");
        close(pipe_fd[1]);
        sleep(3);
        waitpid(pid, NULL, 0);
        exit(0);
    }
}

```

**流管道**
与linux中文件操作有文件流的标准IO一样，管道的操作也支持基于文件流的模式。接口函数如下：
库函数：popen();
原型：``` FILE *open (char *command, char *type); ```
返回值：如果成功，返回一个新的文件流，如果无法创建进程或者管道返回NULL，管道中的数据流方向是由第二个参数进行控制。(相当于管道进程执行了该cmd命令，然后将结果存储在管道中)
```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<fcntl.h>
#define BUFSIZE 1024
int main() {
    FILE *fp;
    char *cmd = "ps -ef";
    char buf[BUFSIZE];
    buf[BUFSIZE] = '\0';
    if ((fp = popen(cmd, "r")) == NULL)
        perror("popen");
    while ((fgets(buf, BUFSIZE, fp)) != NULL)
        printf("%s", buf);
    pclose(fp);
    exit(0);
}
```

**命名管道(FIFO)**
命名管道和一般的管道基本相同，但也有一些显著的不同：
A、命名管道是在文件系统中作为一个特殊的设备文件而存在的。
B、不同祖先的进程之间可以通过管道共享数据。
C、当共享管道的进程执行完所有的IO操作以后，命名管道将继续保存在文件系统中以便以后使用。
管道只能由相关进程使用，它们共同的祖先进程创建了管道。但是，通过FIFO，不相关的进程也能交换数据。
(相当于往一个共享命名管道中进行写读写数据)

**read data from FIFO**
```c
#include<sys/types.h>
#include<sys/stat.h>
#include<errno.h>
#include<fcntl.h>
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#define FIFO "/tmp/myfifo"
main(int argc, char** argv) {
    char buf_r[100];
    int fd;
    int nread;
    if ((mkfifo(FIFO, O_CREAT | O_EXCL) < 0) && (errno != EEXIST))
        printf("cannot create fifoserver\n");
    printf("Preparing for reading bytes....\n");
    memset(buf_r, 0, sizeof(buf_r));
    fd = open(FIFO, O_RDONLY | O_NONBLOCK, 0);
    if (fd == -1) {
        perror("open");
        exit(1);
    }
    while (1) {
        memset(buf_r, 0, sizeof(buf_r));
        if ((nread = read(fd, buf_r, 100)) == -1) {
            if (errno == EAGAIN)
                printf("no data yet\n");
        }
        printf("read %s from FIFO\n", buf_r);
        sleep(1);
    }
    pause();
    unlink(FIFO);
}
```

**write data from FIFO**
```c
#include<sys/types.h>
#include<sys/stat.h>
#include<errno.h>
#include<fcntl.h>
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#define FIFO_SERVER "/tmp/myfifo"
main(int argc, char** argv) {
    int fd;
    char w_buf[100];
    int nwrite;
    if (fd == -1)
        if (errno == ENXIO)
            printf("open error;no reading process\n");
    fd = open(FIFO_SERVER, O_WRONLY | O_NONBLOCK, 0);
    if (argc == 1)
        printf("Please send something\n");
    strcpy(w_buf, argv[1]);
    if ((nwrite = write(fd, w_buf, 100)) == -1) {
        if (errno == EAGAIN)
            printf("The FIFO has not been read yet. Please try later\n");
    } else
        printf("write %s to the FIFO\n", w_buf);
}
```
## 信号
信号是软件中断。信号（signal）机制是Unix系统中最为古老的进程之间的通讯机制。它用于在一个或多个进程之间传递异步信号。很多条件可以产生一个信号。

**kill信号**
```c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<signal.h>
#include<sys/types.h>
#include<sys/wait.h>
int main() {
    pid_t pid;
    int ret;
    if ((pid == fork()) < 0) {
        perror("fork");
        exit(1);
    }
    if (pid == 0) {
        raise(SIGSTOP);
        exit(0);
    } else {
        printf("pid=%d\n", pid);
        if ((waitpid(pid, NULL, WNOHANG)) == 0) {
            if ((ret = kill(pid, SIGKILL)) == 0)
                printf("kill %d\n", pid);
            else {
                perror("kill");
            }
        }
    }
}
```

**alarm.c**
```c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
int main() {
    int ret;
    ret = alarm(5);
    pause();
    printf("I have been waken up.\n", ret);
}
```

**信号的处理**
```c
#include<signal.h>
void (*signal (int signo, void (*func)(int)))(int)
```
返回值：成功则为以前的信号处理配置，若出错则为SIG_ERR

**mysignal.c**
```c
#include<signal.h>
#include<stdio.h>
#include<stdlib.h>
void my_func(int sign_no) {
    if (sign_no == SIGINT)
        printf("I have get SIGINT\n");
    else if (sign_no == SIGQUIT)
        printf("I have get SIGQUIT\n");
}
int main() {
    printf("Waiting for signal SIGINT or SIGQUTI\n");
    signal(SIGINT, my_func);
    signal(SIGQUIT, my_func);
    pasue();
    exit(0);
}
```

**信号集函数组**
我们需要有一个能表示多个信号——信号集（signal set）的数据类型。将在sigprocmask()这样的函数中使用这种数据类型，以告诉内核不允许发生该信号集中的信号。信号集函数组包含水量几大模块：创建函数集、登记信号集、检测信号集。

**创建函数集**
```
#include<signal.h>
int sigemptyset(sigset_t* set);
int sigfillset(sigset_t* set);
int sigaddset(sigset_t* set, int signo );
int sigdelset(sigset_t* set, int signo);
四个函数返回：若成功则为0，若出错则为 -1
int sigismember(const sigset_t* set, int signo);
返回：若真则为1，若假则为0；
signemptyset: 初始化信号集合为空。
sigfillset: 初始化信号集合为所有的信号集合。
sigaddset: 将指定信号添加到现存集中。
sigdelset: 从信号集中删除指定信号。
sigismember: 查询指定信号是否在信号集中。
```

**样例代码**
```C
#include<sys/types.h>
#include<unistd.h>
#include<signal.h>
#include<stdio.h>
#include<stdlib.h>
void my_func(int signum) {
    printf("If you want to quit,please try SIGQUIT\n");
}
int main() {
    sigset_t set, pendset;
    struct sigaction action1, action2;
    if (sigemptyse(&set) < 0)
        perror("sigemptyset");
    if (sigaddset(&set, SIGQUIT) < 0)
        perror("sigaddset");
    if (sigaddset(&set, SIGINT) < 0)
        perror("sigaddset");
    if (sigprocmask(SIG_BLOCK, &set, NULL) < 0)
        perror("sigprcmask");
    esle{
        printf("blocked\n");
        sleep(5);
    }
    /*
    一个进程的信号屏蔽字可以规定当前阻塞而不能递送给该进程的信号集。调用函数sigprocmask可以检测或更改（或两者）进程的信号屏蔽字。
    #include<signal.h>
    int sigprocmask(int how,const sigset_t* set,sigset_t* oset);
    返回：若成功则为0，若出错则为-1
    */
    if (sigprocmask(SIG_UNBLOCK, &set, NULL)
            perror("sigprocmask");
            else
                printf("unblock\n");
        while (1) {
            if (sigismember(&set, SIGINT)) {
                    sigemptyset(&action1.sa_mask);
                    action1.sa_handler = my_func;
                    /*sigaction函数的功能是检查或修改（或两者）与指定信号相关联的处理动作。此函数取代了UNIX早期版本使用的signal函数。
                    #include<signal.h>
                    int sigaction(int signo,const struct sigaction* act,struct sigaction* oact);
                    返回：若成功则为0，若出错则为-1*/
                    sigaction(SIGINT, &action1, NULL);
                } else if (sigismember(&set, SIGQUIT)) {
                    /*sigpending返回对于调用进程被阻塞不能递送和当前未决的信号集。
                    #include<signal.h>
                    int sigpending(sigset_t * set);
                    返回：若成功则为0，若出错则为-1*/
                    sigemptyset(&action2.sa_mask);
                    /*指定信号关联函数，可以是自定义处理函数，还可以SIG_DEF或SIG_IGN;*/
                    action2.sa_handler = SIG_DEL;
                    sigaction(SIGTERM, &action2, NULL);
                }
            }
}
```

## 消息队列
消息队列提供了一种从一个进程向另一个进程发送一个数据块的方法。  每个数据块都被认为含有一个类型，接收进程可以独立地接收含有不同类型的数据结构。我们可以通过发送消息来避免命名管道的同步和阻塞问题。但是消息队列与命名管道一样，每个数据块都有一个最大长度的限制。
Linux用宏MSGMAX和MSGMNB来限制一条消息的最大长度和一个队列的最大[消息队列](https://blog.csdn.net/ljianhui/article/details/10287879)

## 参考资料
        * [总的介绍](http://www.cnblogs.com/linshui91/archive/2010/09/29/1838770.html)
                         * [简介介绍](https://www.pythontab.com/html/2018/linuxkaiyuan_0111/1222.html)
                                 * [消息队列](https://blog.csdn.net/ljianhui/article/details/10287879)
                                         *
