## ubuntu 代理配置

git支持`socks5`和`http`协议，但是ss的linux客户端默认是只有`socks5`的代理端口，通过git来设置`socks5`的代理，不能通过curl来拿到google的html。但是windows的ss客户端能够达到`socks5`和`http`的代理。linux上面要使用工具将http的代理转换为`socks5`的代理。

