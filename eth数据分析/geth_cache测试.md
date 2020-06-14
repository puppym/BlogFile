

geth在启动时关于性能调优的参数如下：

```
PERFORMANCE TUNING OPTIONS:
// 该选项主要是与syncmode有关   light  fast  full
--cache value            Megabytes of memory allocated to internal caching (default: 1024)

// 数据库的缓存
--cache.database value   Percentage of cache memory allowance to use for database io (default: 50)

// 前缀状态树的缓存
--cache.trie value       Percentage of cache memory allowance to use for trie caching (default: 25)

// 前缀状态树裁剪的缓存
--cache.gc value         Percentage of cache memory allowance to use for trie pruning (default: 25)
--trie-cache-gens value  Number of trie node generations to keep in memory (default: 120)
```

![1580871585194](C:\Users\czm18\AppData\Roaming\Typora\typora-user-images\1580871585194.png)

以太坊中的状态裁剪(state pruning)

[ethereum状态裁剪](https://blog.ethereum.org/2015/06/26/state-tree-pruning/)

[ethereum解释](https://ethereum.stackovernet.com/cn/q/63)

> It is a similar concept to garbage collection in programming languages and in tree-based version control system like git. When ethereum contracts run, they modify their state. And since the state tree is an immutable append-only structure, it means that every time the state is modified, it gets a new state root. Some elements that were reachable from the old roots may be not be reachable from the new root (due to operations that delete or modify entries). Theoretically, they can be pruned (garbage collected). However, since the Proof-Of-Work consensus as it is does not define when state transitions are final, there is always a theoretical possibility that state will be reverted to older roots and things that were pruned would be needed again. So, pruning is currently a trade-off. We say, that, for example, after 5000 block we assume that the state won't be reverted and prune all unreachable nodes. Someone might still want to disable this feature to keep the entire history of the blockchain for special purposes.

可以这样理解状态裁剪X=5000，那么最近5000块的状态更改会写到缓存(CacheGCFlag)里面，由于archive模式的节点需要每个区块的世界状态，因此该值设置为0。

对于CacheTrieFlag缓存为执行下一个5000个块之前，需要将当前的世界状态加载进该缓存。

最后一层为level DB

> 上面多级缓存的主要目的是减少在EVM执行交易是频繁直接对磁盘上的数据库levelDB的读写。

                utils.DataDirFlag,
                utils.SyncModeFlag,
                utils.GCModeFlag,
                utils.CacheFlag,
                utils.CacheDatabaseFlag,
                utils.CacheTrieFlag,
                utils.CacheGCFlag,
关于创建合约的交易trace中并没有create记录，详细查看该交易：

0xe0667bb1a212a3c50e8654bbf5170f7ab306743a3e977ae39516bc3ce02b891c

论文->bigquery->fast and full node->finish 和commit。

事件和日志的主要用途有四种：
1.帮助用户客户端（web3.js）读取智能合约的返回值；
2.智能合约异步通知用户客户端（web3.js）；
3.用于智能合约的存储（比Storage便宜得多）；
4.我认为有第四种，可以帮助我们通过filter过滤出历史交易数据，具体实现看下面文章解析。
https://blog.csdn.net/jambeau/article/details/78887216



