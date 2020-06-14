```SQL 
CREATE TABLE user_tbl(name VARCHAR(20), signup_date DATE);

# 执行sql文件
\i /path/sql
psql -d db1 -U userA -f /pathA/xxx.sql

# 插入表数据
INSERT INTO  目标表 (字段1, 字段2, ...)  SELECT   字段1, 字段2, ...   FROM  来源表 ;
INSERT INTO  目标表  SELECT  * FROM  来源表 ;


# 插入数据
INSERT INTO user_tbl(name, signup_date) VALUES('张三', '2013-12-22');

# 查询记录
SELECT * FROM user_tbl;

# 更新数据
UPDATE user_tbl set name = '李四' WHERE name = '张三';

# 删除记录
DELETE FROM user_tbl WHERE name = '李四' ;

# 添加字段
ALTER TABLE user_tbl ADD email VARCHAR(40);

# 更改字段类型
ALTER TABLE user_tbl ALTER COLUMN signup_date SET NOT NULL;

# 设置字段默认值（注意字符串使用单引号）
ALTER TABLE user_tbl ALTER COLUMN email SET DEFAULT 'example@example.com';

# 去除字段默认值
ALTER TABLE user_tbl ALTER email DROP DEFAULT;

# 重命名字段
ALTER TABLE user_tbl RENAME COLUMN signup_date TO signup;

# 删除字段
ALTER TABLE user_tbl DROP COLUMN email;

# 表重命名
ALTER TABLE user_tbl RENAME TO backup_tbl;

# 删除表
DROP TABLE IF EXISTS backup_tbl;

# 删除库
\c hello2;
DROP DATABASE IF EXISTS hello;

# 查询表的索引
select * from pg_indexes where tablename='blocks';

# 分析SQL语句执行效率
explain analyse SQL

# 删除索引
drop index if exists index_name [cascade | restrict]

# ALTER TABLE — 更改一个表的定义

# unix timestamp to timestamp with zone
to_timestamp(unix_timestamp)

# To convert back to unix timestamp you can use date_part:
SELECT date_part('epoch',CURRENT_TIMESTAMP)::int

explain analyse select * from blocks_1_100W where to_timestamp(_timestamp) between '2015-07-31 09:23:00+08' and '2015-08-31 09:23:00+08';

# 获取查询结果行号
ROW_NUMBER()OVER()

# sql分割字符串
split_part(string, split, num=1)

# sql字段自增 

# 查看单个索引大小
select pg_size_pretty(pg_relation_size('idx_join_date_test'));

# 查看一个表的所有索引总大小
select pg_size_pretty(pg_indexes_size('test'));

# 查询所有表的单个索引大小
select indexrelname, pg_size_pretty(pg_relation_size(indexrelid)) from pg_stat_user_indexes where schemaname='public' order by pg_relation_size(indexrelid) desc;

# 查询所有表大小
select indexrelname, pg_size_pretty(pg_relation_size(relid)) from pg_stat_user_indexes where schemaname='public' order by pg_relation_size(relid) desc;

#pg_stat_user_indexes收集统计信息http://www.postgres.cn/docs/9.4/monitoring-stats.html
# 每个表所有索引的大小
select relname, pg_size_pretty(pg_indexes_size(relid)) from pg_stat_user_tables where schemaname='public' order by pg_indexes_size(relid) desc;

# 每个表的总表大小，每个表的数据大小，以及所有索引大小
select relname,pg_size_pretty(pg_total_relation_size(relid)),pg_size_pretty(pg_relation_size(relid)),pg_size_pretty(pg_indexes_size(relid)) from pg_stat_user_tables where schemaname='public' order by pg_indexes_size(relid) desc;

group by， order by 后面跟数字，指的是 select 后面选择的列（属性），1 代表第一个列（属性），依次类推。
```

