## vscode remote 

1. 应用商店搜索remote-develop.
2. new a connection
3. 配置configure，设置ip，端口，Host。
4. 连接，配置端口转发，30000->localhost:30000

### 启动WSL报错权限问题

```
 chmod -R 777 /mnt/c/Users/czm18/.vscode/extensions/ms-vscode-remote.remote-wsl-0
.42.3/
```

### remote config文件配置

对于本地的.ssh中的config文件可以管理多个私钥。[ubuntu管理多个私钥](https://www.jianshu.com/p/fe215c52c534) 然后服务端如果有多个公钥需要将所有的公钥都加入到`authorized_keys`中。 `cat id_rsa.pub > authorized_keys`

