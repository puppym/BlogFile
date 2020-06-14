

### pgsql 基本命令

**安装psycopg2**

```bash
sudo apt-get install libpg-dev
sudo apt-get install python3.X-dev
python -m pip install psycopg2 --user
```



**启动postgresql**

```bash
sudo /etc/init.d/postgresql start  开启
sudo /etc/init.d/postgresql stop  关闭
sudo /etc/init.d/postgresql restart  重启
\password postgres 设置密码
```

默认是postgres用户，因此登录pg数据库

```bash
sudo -u c
su postgres
psql postgres
PGPASSWORD=123456 psql --host 127.0.0.1 postgres -U postgres
```

```sql
PGPASSWORD=123456 pgsql -c "SQL"   // 可以不用输入密码执行pgsql
```

```bash 
select * from pg_tables;

select tablename from pg_tables where schemaname='public';

\d `table_name`    进一步查看表结构
```





**pgsql 执行sql文件**

```bash
**方法一**


pqsql -d db1 -U userA

\i /pathA/XXX.sql

**方法二**

psql -d db1 -U userA -f /path/XXX.sql
```



### python连接postgres将CSV导入数据库

使用`psycopy2`库进行，着重讨论下`commit()`和`rollback()`两个函数。

不同于SQLite，一些对数据库进行修改的SQL首先会被存在一个缓冲区中，只有当执行commit之后，缓冲区中的SQL会被当成一个原子操作被执行并且结果被commit到数据库中，如果原子操作中任意一条语句执行失败都会触发`rollback()`函数，回滚已经执行SQL语句。

举个例子：

```bash
step1:
name  balance
Bob     10
Alice   20


step2:
Alice给Bob转账10,对应到SQL语句如下：
set Bob   Balance 20
set Alice Balance 10

step3:
但是出现下面情况：
set Bob   Balance 20
----- fail ---------
set Alice Balance 10
现在数据库中的数据为：
name  balance
Bob     20
Alice   20

上述情况在实际的生产环境中肯定是不被允许的， 引出了pg里面的commit和rollback。

```



看一个下面正确使用两个函数的例子：[linker]()

```python
import psycopg2
from psycopg2 import Error
try:
   connection = psycopg2.connect(user="postgres",
                                  password="pass@#29",
                                  host="127.0.0.1",
                                  port="5432",
                                  database="postgres_db")
   connection.autocommit=False
   cursor = connection.cursor()
   amount = 2500

   query = """select balance from account where id = 624001562408"""
   cursor.execute(query)
   record = cursor.fetchone()
   balance_account_A  = int(record)
   balance_account_A -= amount

   # Withdraw from account A  now
   sql_update_query = """Update account set balance = %s where id = 624001562408"""
   cursor.execute(sql_update_query,(balance_account_A,))

   query = """select balance from account where id = 2236781258763"""
   cursor.execute(query)
   record = cursor.fetchone()
   balance_account_B = int(record)
   balance_account_B += amount

   # Credit to  account B  now
   sql_update_query = """Update account set balance = %s where id = 2236781258763"""
   cursor.execute(sql_update_query, (balance_account_B,))

   # commiting both the transction to database
   connection.commit()
   print("Transaction completed successfully ")

except (Exception, psycopg2.DatabaseError) as error :
    print ("Error in transction Reverting all other operations of a transction ", error)
    connection.rollback()

finally:
    #closing database connection.
    if(connection):
        cursor.close()
        connection.close()
        print("PostgreSQL connection is closed")
```



进一步改进上面的例子，使用with，从而不显示使用`commit`和`rollback`函数。

```python
with psycopg2.connect(connection_arguments) as conn:
    with conn.cursor() as cursor:
        cursor.execute(Query)
```





### 参考链接

[Python PostgreSQL Transaction management using Commit and Rollback](https://pynative.com/python-postgresql-transaction-management-using-commit-and-rollback/)

[Tutorial: Loading Data into Postgres using Python and CSVs](https://www.dataquest.io/blog/loading-data-into-postgres/)