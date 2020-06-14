- 删除相关的安装

```
sudo apt-get --purge remove postgresql
```

------

- 删除配置及文相关件

```
sudo rm -r /etc/postgresql/
sudo rm -r /etc/postgresql-common/
sudo rm -r /var/lib/postgresql/123
```

------

- 删除用户和所在组

```
sudo userdel -r postgres
sudo groupdel postgres12
```

------

- 重新安装

```
sudo apt-get install postgresql
```

* 修改pg数据目录

```bash
service postgres stop
sudo cp -rf  
默认数据库数据目录/var/lib/postgresql/12/main/ 
vi 
默认配置文件目录/etc/postgresql/9.6/main/postgresql.conf
data_directory='/data/postgresql/datafile'
/mnt/intel-disk/pgdata
chown -R postgres:postgres /mnt/intel-disk/pgdata
chmod 700 /mnt/intel-disk/pgdata
sudo vim postgres.conf
data_directory = '/mnt/intel-disk/pgdata'
service postgres restart
sudo /etc/init.d/postgresql restart

查看当前数据目录
sudo -u postgres psql -c "show data_directory"

https://www.cnblogs.com/easonjim/p/9052836.html
https://mozillazg.com/2014/06/hello-postgresql.html
```

* 修改pg用户和database密码

安装pg后会新建一个postgres的用户和database，修改配置文件做到局域网外部能够连接：

```bash
# 设置密码
sudo passwd postgres
sudo -u postgres psql
\d  查看表
\l   查看数据库
\d tablename 查看表结构
\password           设置密码。
\q                  退出。
\h                  查看SQL命令的解释，比如\h select。
\?                  查看psql命令列表。
\l                  列出所有数据库。
\c [database_name]  连接其他数据库。
\d                  列出当前数据库的所有表格。
\d [table_name]     列出某一张表格的结构。
\du                 列出所有用户。
\e                  打开文本编辑器。
\conninfo           列出当前数据库和连接的信息。
\i                  执行sql文件
\password postgres
\q
service postgres restart

# 更改配置
vim postgresql.conf
listen_addresses = '*'
password_encryption = on
vim /etc/postgresql/12/main/pg_hba.conf
host    all             all             0.0.0.0/0               md5
# 允许任何人能够通过MD5加密的方式进行访问
# http://www.postgres.cn/docs/11/auth-pg-hba-conf.html
sudo /etc/init.d/postgresql restart

```





