# socket 
主机字节序是高高低低，网络字节序是高低高低。 
在linux中，一切皆文件。普通的文本文件确实是文件，但是设备，socket，管道等都被当成文件处理。所以我们获得的connfd也是一个文件描述符。在linux的文件描述符中有三个是特殊的，任何进程都一样，0,1,2分别代表的是标准输入，标准输出和标准出错。

AF = Address Family，PF = Protocol Family，所以最好在指示地址的时候使用AF，在指示协议的时候使用PF。因为那时人们希望同一个地址族（ "AF" in "AF_INET" )可能支持多个协议族 ("PF" in "PF_INET" )。这样的话，就可以加以区分。

# raw_socket 
packet_socket = socket(AF_PACKET, int socket_type, int protocol); 
AF_INET, PF_INET(两者表示相同的值), AF_PACKET和PF_PACKET。NET和PACKET的区别是AF_INET是面向IP层的原始套接字，AF_PACKET是面向数据链路层的套接字。  
AF_INET的第三个参数IPPROTO_TCP、IPPROTO_UDP、IPPROTO_ICMP和IPPROTO_RAW 

AF_PACKET的第三个参数  
ETH_P_IP - 只接收目的mac是本机的IP类型数据帧 
ETH_P_ARP - 只接收目的mac是本机的ARP类型数据帧 
ETH_P_RARP - 只接收目的mac是本机的RARP类型数据帧 
ETH_P_PAE - 只接收目的mac是本机的802.1x类型的数据帧 
ETH_P_ALL -接收目的mac是本机的所有类型数据帧，同时还可以接收本机发出的所有数据帧，混杂模式打开时，还可以接收到目的mac不是本机的数据帧 

icmp报文请求是8，回应是0。

## 安全问题：

如果需要对所有使用rawsocket的程序都设置root访问权限，那么有可能程序可以对系统造成严重损害，特别足够复杂的网络程序，可能很容易出现错误。

Linux Capabilities

为了解决root权限问题，Linux推出了capability这一功能，程序不必以完全root访问权限运行。简而言之，capability是对root权限的细分。譬如raw socket就只需要root权限中很少一部分功能，也即CAP_NET_RAW。这样的话，以前仅限于以root用户身份运行的进程（使用sudo）的某些功能由非特权进程使用，只要相关的capability打开就行。

使用setcap命令在每个文件的基础上设置相应的capability。此示例将CAP_NET_RAW和CAP_NET_ADMIN功能应用于a.out二进制文件。 一旦在文件上设置了这些功能，非root用户就可以运行这些程序。

$ sudo setcap cap_net_admin,cap_net_raw=eip a.out
可以检查一下：

$ sudo getcap a.out
a.out = cap_net_admin,cap_net_raw+eip