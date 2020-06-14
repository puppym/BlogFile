## swap 常见操作

```bash
➜  ~ sudo swapon -s
Filename                Type        Size    Used    Priority
/swapfile                               file        2097148 592028  -1
➜  ~ sudo swapoff /swapfile 
➜  ~ sudo fallocate -l 16G /swapfile
➜  ~ sudo mkswap /swapfile
mkswap: /swapfile: warning: wiping old swap signature.
Setting up swapspace version 1, size = 16 GiB (17179865088 bytes)
no label, UUID=f8e26399-d888-4907-b91b-a426027154e0
➜  ~ sudo swapon /swapfile

sudo swapoff -a #禁用swap命令
sudo swapon -a #启用命令
sudo free -m #查看交换分区
```

### 永久禁用swap

```bash
sudo mount -n -o remount,rw /  # 把根目录文件系统设为可读写
vi /etc/fstab  # 用vi修改/etc/fstab文件，在swap分区这行前加 # 禁用掉，保存退出
reboot
sudo free -m
```

