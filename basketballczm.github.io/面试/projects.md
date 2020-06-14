# simple_DSP 
## redis 
select num  选择第几号数据库

## hash表 
hset key field value  hashset中，一个key对应多个域，一个域对应一个value
hgetall key 获取该hash表中的所有域和key，并且以列表的形式返回hash表的域和域的值

## 字符串 
set key value 将一个字符串的值关联到key
get key       返回与键相互关联的字符串值

## 有序集合 
zadd key score member[score member ... ] 将一个或者多个member元素及其score值加入到有序集key当中。score值可以是整数值或者双精度浮点数
zrangebyscore key min max [withscores]  默认情况下，区间的取值使用闭区间 (小于等于或大于等于)，你也可以通过给参数前增加 ( 符号来使用可选的开区间 (小于或大于)。取出在这个区间内的所有有序集合
zrange key start stop [withscores] 返回有序集key中，指定区间内带有score值(可选)的有序集成员的列表，其中成员的位置按score值递增排序，start和stop分别是指下标。

## 集合
sadd key member[member...]将一个或者多个member元素加入到集合key当中，已经存在于集合的member元素将被忽略，假如key不存在，则创建一个只包含member元素成员的集合。
semebers key 返回集合key中的所有成员，不存在的key被视为空集合。






