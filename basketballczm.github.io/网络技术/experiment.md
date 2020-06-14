# 实验1
## ARP
显示和修改与物理地址之间的转换表
arp -a    显示ip和物理地址及之间的关系
添加ip到物理地址的转换关系表，Win7下绑定IP和MAC地址操作和XP有所差别，Win7用户这时候就需要用netsh命令了。具体操作如下：
netsh ii show in  查看idx(在本机上看到了本地连接和无线网络连接和一些虚拟的网卡，注意区分他们的idx)
netsh -c "i i" add neighbors idx 157.55.85.212  00-aa-00-62-c6-09

# 实验2
## wireshark捕获ping包并且解释
Display Filters添加一个过滤项，并且选择这个过滤项。

1. Frame Length: 74 bytes (592 bits)  ICMP的整个长度使74字节
2. Ethernet II, Src: CompalIn_13:1e:3d (f0:76:1c:13:1e:3d), Dst: Hangzhou_71:30:02 (50:da:00:71:30:02) 前面14个字节是以太网的帧头
Destination: Hangzhou_71:30:02 (50:da:00:71:30:02) 前6个字节
Source: CompalIn_13:1e:3d (f0:76:1c:13:1e:3d) 6字节源和目的的物理地址
Type: IPv4 (0x0800) 协议类型
3. Internet Protocol Version 4, Src: 219.219.220.235, Dst: 180.97.33.108 IP头部 20字节
.... 0101 = Header Length: 20 bytes (5) 头部长度是20字节
Total Length: 60 总长度为60字节
Source: 219.219.220.235 源IP地址
Destination: 180.97.33.108 目的IP地址
4. Internet Control Message Protocol 40字节的ICMP报文
前8个字节是ICMP头部，后面32字节是ICMP数据部分。ICMP信息头包括：类型（8或0）、代码（0）、检验和、标示符、序号。
Type: 8 (Echo (ping) request)请求报文类型为8，应答报文类型为0 1字节
Code: 0   1字节
Checksum: 0x4cc9 [correct] 2字节
Identifier (BE): 1 (0x0001) 2字节
Sequence number (BE): 146 (0x0092) 2字节

## tracert
在Wireshark下，使用Tracert程序捕获ICMP分组，该程序能够映射出通往特定的因特网自己途径的所有中间主机。
源端发送一串ICMP分组到目的端，发送第一个分组时，TTL=1；发送第二个分组时TTL=2，依次类推。路由器把经过它的每一个分组TTL字段值减1，当一个分组到达了路由器时，TTL字段的值为1时，路由器会发送一个ICMP差错分组给源端。

1. Frame Length: 106 bytes (848 bits) 
2. Ethernet II, Src: CompalIn_13:1e:3d (f0:76:1c:13:1e:3d), Dst: Hangzhou_71:30:02 (50:da:00:71:30:02) 前面14个字节是以太网帧头
3. Internet Protocol Version 4, Src: 219.219.220.235, Dst: 180.97.33.107 IP报头20个字节
4. Internet Control Message Protocol 40字节的ICMP报文
前8个字节是ICMP头部，后面32字节是ICMP数据部分。ICMP信息头包括：类型（8或0）、代码（0）、检验和、标示符、序号。
Type: 8 (Echo (ping) request)请求报文类型为8，应答报文类型为0 1字节
Code: 0   1字节
Checksum: 0x4cc9 [correct] 2字节
Identifier (BE): 1 (0x0001) 2字节
Sequence number (BE): 146 (0x0092) 2字节

ICMP协议主要是通过Type和Code来识别的8,0表示报文为诊断报文和请求测试包。0,0表示应答报文
注意各个ICMP报文的time to live 的值是不相同的。
数据部分前者是32个字节，后者是64个字节


## FTP下载文件
TCP3次握手的建立标志位

- -> SYN=1 seq=x
- <- SYN=1,ACK=1,seq=y,ack=x+1
- -> ACK=1,seq=x+1,ack=y+1

TCP 4次挥手


# 实验3
1. 交换机能够通过端口划分不同的vlan，然后使得相同的vlan能够通讯，不同的vlan不能进行通讯。
2. 关于网关，许多有关TCP/IP的文献曾经把网络层使用的路由器称为网关，在今天很多局域网采用都是路由器来接入网络，因此通常指的网关就是路由器的IP。一般来说路由器的LAN接口的IP地址就是你所在局域网中的网关，当你所在的局域网的计算机需要和其它局域网中的计算机，或者需要访问互联网的时候，你所在局域网的计算机机会先把数据包传输到网关(路由器的LAN接口)，然后再由网关进行转发。https://blog.csdn.net/haifengid/article/details/51537914
