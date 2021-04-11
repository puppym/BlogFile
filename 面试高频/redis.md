## redis

redis 事务

Redis 通过 [MULTI](http://redis.readthedocs.org/en/latest/transaction/multi.html#multi) 、 [DISCARD](http://redis.readthedocs.org/en/latest/transaction/discard.html#discard) 、 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 和 [WATCH](http://redis.readthedocs.org/en/latest/transaction/watch.html#watch) 四个命令来实现事务功能， 本章首先讨论使用 [MULTI](http://redis.readthedocs.org/en/latest/transaction/multi.html#multi) 、 [DISCARD](http://redis.readthedocs.org/en/latest/transaction/discard.html#discard) 和 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 三个命令实现的一般事务， 然后再来讨论带有 [WATCH](http://redis.readthedocs.org/en/latest/transaction/watch.html#watch) 的事务的实现。

因为事务的安全性也非常重要， 所以本章最后通过常见的 ACID 性质对 Redis 事务的安全性进行了说明。

https://redisbook.readthedocs.io/en/latest/feature/transaction.html