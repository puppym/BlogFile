* ```shell
  ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
  ```

* 传送公钥上服务器

* ssh -T git@github.com 查看是否成功

注意此时可能还是需要输入账号密码，继续将私钥添加进

* ```shell
  eval $(ssh-agent -s)
  ```

* ```shell
  ssh-add ~/.ssh/id_rsa
  ```

### 参考文档

[github官方资料](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)