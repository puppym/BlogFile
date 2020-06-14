# netfilter 简介 
Netfilter是Linux 2.4内核中的子系统。Netfilter通过使用内核网络代码中的各种钩子（hook），实现了包过滤、网络地址转换（NAT）和连接跟踪等网络技巧。这些钩子是内核代码（静态构建或以可加载模块的形式）可以注册要为特定网络事件调用的函数的位置。这种事件的一个例子是接收分组。

# netfilter 包过滤 
通过加载netfilter包过滤的内核模块，通过过滤指定的数据包，然后根据一定格式获取数据包中的一些信息。比如首先过滤出请求端口号为21，再根据匹配每个数据包的数据串"USER "和"PASSWD "来过滤数据包中的账户和密码

# 内核模块命令 
内核模块驱动的基本命令 

dmesg | tail -n //查看内核模块的打印信息 
insmod \*.ko    //插入内核模块 
modprobe        //插入驱动，不需要加.ko后缀
lsmod           //查看内核模块 
rmmod           //删除内核模块 
modinfo         //获取模块的信息 

# netfilter攻击流程 
1. 会用wireshark工具进行抓包，抓取ftp包，分析ftp数据包的格式，确定要如何解析出USER和PASSWD。(每一类协议USER和PASSWD的传输过程中，字符串的格式不同)。 
2. 首先watch_out函数在postRoute阶段对ftp的包进行过滤，获取ftp的所有数据包。 
3. check_ftp对抓取到的ftp数据包数据部分进行解析，针对上面的分析结果使用特定的代码解析出USER和PASSWD，并且将获取到的USER和PASSWD保存在内核空间中。 
4. watch_in函数是对preRoute阶段的数据包进行过滤，获取到MAGIC的数据包，然后回应特定的MAGIC数据包，并且将内核中保存的账号和密码拷贝到MAGIC应答数据包中。 
5. getpass模块接收到MAGIC数据包的应答数据包，解析应答数据包，即可获取所窃取的账号或者密码。 

# 注意事项 

sudo find /usr/src -name \*.h | xargs grep -rn iphdr
-r 是递归查找
-n 是显示行号
-R 查找所有文件包含子目录
-i 忽略大小写
