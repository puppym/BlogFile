ICMP重定向信息是路由器向主机提供实时的路由信息，当一个主机收到ICMP重定向信息时，它就会根据这个信息来更新自己的路由表。由于缺乏必要的合法性检查，如果一个黑客想要被攻击的主机修改它的路由表，黑客就会发送ICMP重定向信息给被攻击的主机，让该主机按照黑客的要求来修改路由表。

## ICMP重定向攻击步骤

1. 嗅探ICMP请求报文。
2. 攻击者通过冒充网关向受害者发送ICMP重定向报文，更改受害主机的默认网关。 

## 注意事项 
1. 捕获嗅探包的时候socket参数使用socket(AF_INET, SOCK_RAW, IPPROTO_ICMP)，IPPROTO_ICMP是因为在构造ICMP重定向包的时候需要28字节的：收到IP数据报的一部分，包括IP首部以及数据报数据的前8个字节，因此要解ICMP数据报。(这里数据传输是ICMP协议)
setsockopt(sockfd, IPPROTO_IP, IP_HDRINCL, ptr_one, sizeof(one));这里参数改为IPPROTO_IP是因为重定向数据报中要填充收到IP数据报的一部分，包括IP首部以及数据报数据的前8个字节。(这里的数据传输是IP协议)。两者都是网络层的协议，ip用于网际互联，icmp用于探测和差错控制。
 
## 参考链接

* https://www.cnblogs.com/gejuncheng/p/7703604.html