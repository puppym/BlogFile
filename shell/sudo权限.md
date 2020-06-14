# [ubuntu为用户增加sudoer权限的两种方法](https://www.cnblogs.com/jiangz/p/4183461.html)



方法一、使用usermod命令

新增user

sudo adduser username

增加sudo权限

sudo usermod -aG sudo username

[sudo usermod -aG sudo ](http://askubuntu.com/questions/2214/how-do-i-add-a-user-to-the-sudo-group)

方法二、修改/etc/sudoers文件

修改文件前先开通root

具体方法是：[Ubuntu技巧之 is not in the sudoers file解决方法_Linux教程_Linux公社-Linux系统门户网站](http://www.linuxidc.com/Linux/2010-12/30386.htm)

修改文件前先开通root，如果没看通，问题很严重。就是修改了/etc/sudoer权限之后再改不回去了，导致当前用户的sudo权限也没有了，需要进入recovery模式修复。见[ubuntu手贱改了sudoers权限之后的恢复 - piaomiao1314 - 博客园](http://www.cnblogs.com/piaomiao1314/p/sudoers.html)

 

修改后输入sudo apt-get update可以确认获得sudo权限