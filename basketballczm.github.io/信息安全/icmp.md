# icmp协议说明 
有多种不同的ICMP报文，每种报文都有自己的格式，但是所有的ICMP报文都有三个共同的字段：

TYPE(8-bit): identifies the message
CODE(8-bit): provides further information about the message type(TYPE = 3, CODE字段说明具体不可达到类型)
CHECKSUM(16-bit)
In addition, ICMP messages that report errors always include the header and the first 64 data bits of the datagram causing the problem.
ICMP的消息类型：TYPE

0:Echo Reply
3:Destination Unreachable
4:Source Quench
5:Redirect (change route)
8:Echo Request
9:Router Advertisement
10:Router Solicitation
11:time Exceeded for a Datagram
12:Parameter Problem on a Datagram
13:timestamp Request
14:Timestamp Reply
17:Address Mask Request
18:Address Mask Reply
值得注意的是，ICMP ECHO （也即我们所熟悉的ping），其中类型是0，是回复；类型是8，是请求。ping用于探测主机的可达性。

ICMP重定向报文 TYPE1字节 CODE1字节 checksum2字节 Geteway Internet Address4字节加28个字节的IP头部以及数据 

# smurf Attack 
smurf Attack是一种分布式拒绝服务攻击，其中使用IP广播地址将具有预期受害者的欺骗源IP的大量互联网控制消息协议（ICMP）分组广播到计算机网络。 默认情况下，网络上的大多数设备都会通过向源IP地址发送回复来对此做出响应。 如果网络上接收和响应这些数据包的机器数量非常大，受害者的计算机将忙于处理ping回复包。 这可能会使受害者的计算机变慢，无法继续工作。

**防御方式**1. 配置各个主机和路由器不响应ICMP请求或广播。2.配置路由器不转发定向到广播地址的数据包。 
