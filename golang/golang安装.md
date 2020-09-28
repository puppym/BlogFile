## golang简单安装

* [简书](https://www.jianshu.com/p/c43ebab25484)
* [官网教程](https://golang.org/doc/install?download=go1.13.4.linux-amd64.tar.gz)

wget直接拉高版本的go好像报错，可以直接拉1.11。

[教程将http代理转化为sock5代理，并且在本地设置PAC模式](http://xiezhongzhao.top/2017/12/18/Ubuntu%E4%B8%8B%E8%AE%BE%E7%BD%AEShadowsocks%E7%9A%84%E9%9D%9E%E5%85%A8%E5%B1%80%E4%BB%A3%E7%90%86(PAC%E8%87%AA%E5%8A%A8%E4%BB%A3%E7%90%86)/)

[翻墙安装一些插件](https://ibz.bz/2019/01/05/f2f7b7e5fdadeaa7ea1b557f60a814de.html)

[上面链接delve库因为库版本过高报错](https://github.com/go-delve/delve/issues/1651)

 google安装插件需要进行翻墙

```bash
# ubuntu 子系统按照下面安装go会报错rename permission denied。因此子系统不要安装在/usr/local/go 下面，直接安装在/home/czm/goroot https://stackoverflow.com/questions/58096783/visual-studio-code-cant-install-the-go-tools
wget https://dl.google.com/go/go1.13.4.linux-amd64.tar.gz
tar -C /usr/local -zxvf  go1.13.4.linux-amd64.tar.gz
sudo tar -C /usr/local -zxvf  go1.13.4.linux-amd64.tar.gz
sudo vim /etc/profile
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
export GO111MODULE=on
export GOPATH=/home/czm/gopath
export GOPROXY=https://goproxy.io
source /etc/profile
go env

cd go && mkdir bin pkg src

##vscode在打开go项目是，打开执行项目然后再go get，下载第三方依赖包，不然会报错`go get cannot load github.com/ethereum/go-ethereum/accounts: rename permission denied`

sudo apt-get autoremove gcc
sudo apt-get autoremove g++
sudo apt-get autoremove python-pip

sudo vim /etc/apt/sources.list

deb https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse

sudo apt-get update
sudo apt-get upgrade

# 因为上面跟新了源，因此直接sudo apt-get install gcc-7
sudo apt-get install gcc-7

# 使用自己的仓库
replace =>
GOPRIVATE="gitlab.com/idmabar,bitbucket.org/idmabar,github.com/idmabar"

# 更换go get 默认使用https去拉去库，换成ssh
git config --global url."git@gitee.com:rockyang/testmod.git".insteadOf "https://gitee.com/rockyang/testmod.git"

# go tags解释

```

### go tags

前有过gopkg.in+go get这种解决方案，而新的go get所支持的版本选择则是这一方案的进一步扩展，看几条规则：

- go get会自动下载并安装package，然后更新到go.mod中
- 可以使用go get package[@version]来安装指定版本的package，不指定version时默认行为和go get package@latest一样
- version可以是vx.y.z这种形式或者直接使用commit的checksum，也可以是master或者latest
- 当version是latest时，也就是默认行为，对于有tags的package，会选取最新的tag，对于没有tags的package，则选取最新的commit
- 当version是master时，不管package有没有打tag，都会选择master分支的最新commit
- 可以在version前使用>，>=，<，<=，表示选取的版本不得超过/低于version，在这个范围内的符合latest条件的版本
- go get -u可以更新package到latest版本
- go get -u=patch将只更新小版本，例如从v1.2.4到v1.2.5
- 当想要修改package的版本时，只需要go get package@指定的version即可



#### 包的版本控制

下面的版本都是合法的：

```
gopkg.in/tomb.v1 v1.0.0-20141024135613-dd632973f1e7
gopkg.in/vmihailenco/msgpack.v2 v2.9.1
gopkg.in/yaml.v2 <=v2.2.1
github.com/tatsushid/go-fastping v0.0.0-20160109021039-d7bb493dee3e
latest
```

版本号遵循如下规律：

```
vX.Y.Z-pre.0.yyyymmddhhmmss-abcdefabcdef
vX.0.0-yyyymmddhhmmss-abcdefabcdef
vX.Y.(Z+1)-0.yyyymmddhhmmss-abcdefabcdef
vX.Y.Z
```

也就是版本号 + 时间戳 + hash，我们自己指定版本时只需要制定版本号即可，没有版本 tag 的则需要找到对应 commit 的时间和 hash 值。

另外版本号是支持 query 表达式的，其求值算法是 “选择最接近于比较目标的版本 (tagged version)”，即上文中的 [gopkg.in/yaml.v2](http://gopkg.in/yaml.v2) 会找不高于 v2.2.1 的最高版本。

对于复杂的包依赖场景，可以参考 Russ Cox 在 [“Minimal Version Selection”](https://research.swtch.com/vgo-mvs) 一文中给出的形象的算法解释 (注意：这个算法仅是便于人类理解，但是性能低下，真正的实现并非按照这个算法实现)。

### GOPROXY 环境变量

终于到了本文的终极大杀器 —— **GOPROXY**。

我们知道从 `Go 1.11` 版本开始，官方支持了 `go module` 包依赖管理工具。

其实还新增了 `GOPROXY` 环境变量。如果设置了该变量，下载源代码时将会通过这个环境变量设置的代理地址，而不再是以前的直接从代码库下载。这无疑对我等无法科学上网的开发良民来说是最大的福音。

更可喜的是，[goproxy.io](https://github.com/goproxyio/goproxy) 这个开源项目帮我们实现好了我们想要的。该项目允许开发者一键构建自己的 `GOPROXY` 代理服务。同时，也提供了公用的代理服务 `https://goproxy.io`，我们只需设置该环境变量即可正常下载被墙的源码包了：

| `1 ` | export GOPROXY=https://goproxy.io |
| ---- | --------------------------------- |
|      |                                   |

不过，**需要依赖于 go module 功能**。可通过 `export GO111MODULE=on` 开启 MODULE。

如果项目不在 `GOPATH` 中，则无法使用 `go get ...`，但可以使用 `go mod ...` 相关命令。

也可以通过置空这个环境变量来关闭，`export GOPROXY=`。

对于 Windows 用户，可以在 `PowerShell` 中设置：

| `1 ` | `$env:GOPROXY="https://goproxy.io"` |
| ---- | ----------------------------------- |
|      |                                     |

最后，我们当然推荐使用 `GOPROXY` 这个环境变量的解决方式，前提是 **Go version >= 1.11**。

最后的最后，七牛也出了个国内代理 [goproxy.cn](https://github.com/goproxy/goproxy.cn) 方便国内用户更快的访问不能访问的包，真是良心。

[go代理不用翻墙解决](https://shockerli.net/post/go-get-golang-org-x-solution/)





### 新建一个自己的项目该怎么做



go-ethereum 项目放在`$GOPATH/src/github/ethereum` 下面go install 之后，查看代码时能够正常查看追溯代码。同理一般的项目也可以放在`$GOPATH/src`下面，如果不放在下面也可以对`$GOPATH`宏指定多条路径。



### 查看库的API接口

godoc 工具需要自己从`golang.org/x/tools/cmd/godoc`下面自己手动安装，`godoc`默认读取`$GOROOT`下面的系统库和第三方库(`github.com`和`golang.org`)。如果`$GOROOT`没有设置就读取`$GOPATH`下面的路径。这样不重新在启动godoc时设定goroot的值，就读取不到安装在`$GOPATH`下面的库。暂时的解决方法是查看系统库用root，查看第三方库用path，也可以将path中的第三方库拷贝到root中，直接在root中查看所有的系统库和第三方库。

### golang一些语法

#### def 关键字

```golang
关键字 defer 允许我们进行一些函数执行完成后的收尾工作，例如：
1. 关闭文件流：
// open a file defer file.Close() （详见第 12.2 节）
2. 解锁一个加锁的资源
mu.Lock() defer mu.Unlock() （详见第 9.3 节）
3. 打印最终报告
printHeader() defer printFooter()
4. 关闭数据库链接
// open a database connection defer disconnectFromDB()
```

#### go 关键字

```golang
/**
 * 每次调用方法会新建一个 channel : resultChan，
 * 同时新建一个 goroutine 来发起 http 请求并获取结果。
 * 获取到结果之后 goroutine 会将结果写入到 resultChan。
 */
func UnblockGet(requestUrl string) chan string {
    resultChan := make(chan string)
    go func() {
        request := httplib.Get(requestUrl)
        content, err := request.String()
        if err != nil {
            content = "" + err.Error()
        }
        resultChan <- content
    } ()
    return resultChan
}
```

外部不需要等待resultChan的值，相当于起一个协程计算其值，然后主线程并不会一直等待resultChan的返回。

#### chan 跨协程通讯

通常使用这样的格式来声明通道： var identifier chan datatype
未初始化的通道的值是nil。通道也是引用类型，所以我们使用 make() 函数来给它分配内存。这里先声明了一个字符串通道 ch1，然后创建了它（实例化）：

```golang
var ch1 chan string
ch1 = make(chan string)
```

当然可以更短： ch1 := make(chan string) 。

chan 被关闭之后`<-ch1`就不会再进行阻塞了。

```golang
package main
  
import ("fmt"
		"time")

func main() {
        ch1 := make(chan int, 8)
        go pump(ch1)       // pump hangs
        fmt.Println(<-ch1) // prints only 0
        for i := 10; i >0; i-- {
                fmt.Println("1")
                <-ch1
                fmt.Println("2")
        }
        fmt.Println("end......")
        //time.Sleep(1e9)
}
func pump(ch chan int) {
        time.Sleep(1e9 * 5)
        close(ch)
}
```



#### go 方法

```golang
func (recv receiver_type) methodName(parameter list) (return value list){...}

如果不需要recv的值，可以使用如下方法替换:
func (_ receiver_type) methodName(parameter list) (return value list){...}

(recv receiver_type)就像定义类成员函数中的类域，recv相当于面向对象中的self和this指针，但是Go中没有这两个关键字，但是可以更具个人喜好使用self和this来代替recv。
```

#### os/signal

- SIGBUS（总线错误）, SIGFPE（算术错误）和 SIGSEGV（段错误）称为同步信号，它们在程序执行错误时触发，而不是通过 `os.Process.Kill` 之类的触发。通常，Go 程序会将这类信号转为 run-time panic。
- SIGHUP（挂起）, SIGINT（中断）或 SIGTERM（终止）默认会使得程序退出。
- SIGQUIT, SIGILL, SIGTRAP, SIGABRT, SIGSTKFLT, SIGEMT 或 SIGSYS 默认会使得程序退出，同时生成 stack dump。
- SIGTSTP, SIGTTIN 或 SIGTTOU，这是 shell 使用的，作业控制的信号，执行系统默认的行为。
- SIGPROF（性能分析定时器，记录 CPU 时间，包括用户态和内核态）， Go 运行时使用该信号实现 `runtime.CPUProfile`。
- 其他信号，Go 捕获了，但没有做任何处理。

#### new 和 make

`new` 的作用是 初始化 一个指向类型的指针 (*T)， make 的作用是为 `slice`, `map` 或者 `channel` 初始化，并且返回引用 T

`make(T, args)`函数的目的与`new(T)`不同。它仅仅用于创建 Slice, Map 和 Channel，并且返回类型是 T（不是T*）的一个初始化的（不是零值）的实例。 这中差别的出现是由于这三种类型实质上是**对在使用前必须进行初始化的数据结构**的**引用**。 例如, Slice是一个 具有三项内容的描述符，包括 指向数据（在一个数组内部）的指针，长度以及容量。**在这三项内容被初始化之前，Slice的值为nil**。对于Slice，Map和Channel， make（）函数初始化了其内部的数据结构，并且准备了将要使用的值。

**问题：栈中分配的变量，出栈的时候不会被释放吗？**

我们注意到有如下函数：

```
    func newInt *int {
        var i int
        return &i   //为何可以返回局部变量呢？
    }
    someInt := newInt()
```

可能golang使用引用技术的GC方法，即使是在函数内部alloc的返回出去编译器也会将这个对象的引用计数+1.

#### os.OpenFile 参数

```golang
const (
    O_RDONLY int = syscall.O_RDONLY // 只读模式打开文件
    O_WRONLY int = syscall.O_WRONLY // 只写模式打开文件
    O_RDWR   int = syscall.O_RDWR   // 读写模式打开文件
    O_APPEND int = syscall.O_APPEND // 写操作时将数据附加到文件尾部
    O_CREATE int = syscall.O_CREAT  // 如果不存在将创建一个新文件
    O_EXCL   int = syscall.O_EXCL   // 和 O_CREATE 配合使用，文件必须不存在
    O_SYNC   int = syscall.O_SYNC   // 打开文件用于同步 I/O
    O_TRUNC  int = syscall.O_TRUNC  // 如果可能，打开时清空文件
)
```

#### interface{}

I want to parse a JSON file to a `map[string]interface{}`:

```golang
var migrations map[string]interface{}
json.Unmarshal(raw, &migrations)

fmt.Println(migrations["create_user"])
```

But I modified my code to point data to `interface{}`:

```golang
var migrations interface{}
json.Unmarshal(raw, &migrations)

// compile wrong here
fmt.Println(migrations["create_user"])
```

```golang
fmt.Println(migrations.(interface{}).(map[string]interface{})["create_user"])

For an expression x of interface type and a type T, the primary expression
x.(T)
```