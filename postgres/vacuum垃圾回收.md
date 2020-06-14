#### 介绍

数据库总是不断地在执行删除，更新等操作。良好的空间管理非常重要，能够对性能带来大幅提高。在postgresql中用于维护数据库磁盘空间的工具是VACUUM，其重要的作用是删除那些已经标示为删除的数据并释放空间。 postgresql中执行delete,update操作后，表中的记录只是被标示为删除状态，并没有释放空间，在以后的update或insert操作中该部分的空间是不能够被重用的。经过vacuum清理后，空间才能得到释放。

#### 意义

PostgreSQL每个表和索引的数据都是由很多个固定尺寸的页面存储（通常是 8kB，不过在编译服务器时[–with-blocksize]可以选择其他不同的尺寸）

PostgreSQL中数据操作永远是Append操作,具体含义如下:

- insert 时向页中添加一条数据
- update 将历史数据标记为无效,然后向页中添加新数据
- delete 将历史数据标记为无效

因为这个特性,所以需要定期对数据库vacuum,否则会导致数据库膨胀,建议打开autovacuum.

#### 文法

```
VACUUM [ ( { FULL | FREEZE | VERBOSE | ANALYZE | DISABLE_PAGE_SKIPPING } [, ...] ) ] [ table_name [ (column_name [, ...] ) ] ]
VACUUM [ FULL ] [ FREEZE ] [ VERBOSE ] [ table_name ]
VACUUM [ FULL ] [ FREEZE ] [ VERBOSE ] ANALYZE [ table_name [ (column_name [, ...] ) ] ]
```