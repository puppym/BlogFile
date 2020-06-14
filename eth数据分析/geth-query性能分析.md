## geth-query性能分析

### geth-query

当前geth-query重放合约速度很慢，为了具体定位导致geth-query工具重放合约慢的原因，特做一下分析，主要从四个方面分析geth-query运行合约时间：1. geth关闭debug运行合约消耗时间。2. geth-query关闭debug运行合约消耗时间。3. geth-query打开debug运行合约消耗时间。4. geth-query打开debug并且过滤opcode运行合约消耗的时间。下面主要统计（这里统计实验平台在小米笔记本本地进行）选取1378000-1380000进行统计，下面测量过程中多次测量取中值:

| kind                                      | blocks          | all block process time                                       |
| ----------------------------------------- | --------------- | ------------------------------------------------------------ |
| geth关闭debug运行合约                     | 1379144         | 这里由于第一次同步没有充足日志，导致不能统计总时间，但是该部分与表第二项有一定差距 |
| geth-query关闭debug运行合约               | 1378000-1380000 | all processBlocks 10.4284397s                                |
| geth-query打开debug运行合约               | 1378000-1380000 | all processBlocks 12.1530857s                                |
| geth-query打开debug并且过滤opcode运行合约 | 1378000-1380000 | all processBlocks 10.4400457s                                |

通过上表发现第二项和第四项数值相差不大，可能是因为内部交易的transfer较少的缘故，因此针对这两项，在服务器上取后面的块进行测试，结果如下：

| kind                                      | blocks         | all block process time                     |
| ----------------------------------------- | -------------- | ------------------------------------------ |
| geth-query关闭debug运行合约               | 6899800-690000 | 19.196630592s.19.257768929s.               |
| geth-query打开debug并且过滤opcode运行合约 | 6899800-690000 | 21.534720076s.21.459072717s.21.324969806s. |

因此上面的测试结果满足假设



通过第一个表，发现geth-query和geth在同时关闭debug的情况下，其运行合约时间差异(不包括geth每次跑完交易之后往数据库写数据花费的时间)，因此下文重点分析了geth-query关闭debug各个操作消耗的时间，定位时间消耗的位置。具体分析如下(实验平台为服务器03)：

通过上面的结果，分析出geth-query在以下几个方面消耗的时间比较多：

1. 由交易签名反推from地址的过程，比较消耗时间，这一过程通过查看源代码，发现每笔交易只执行一次。
2. newTransactionTracer函数中对于账户的balance，首先会从stateDB中去读balance，如果state DB中不存在该账户的余额，然后会从底层levelDB的MPT树种读取balance，因此这里读数据库的操作比较耗时。
3. 由于执行transaction的字节码的函数种去除了一些commit数据库的操作，因此geth-query交易执行的 速度会快于geth交易执行速度。

后面的阶段的测试主要是想测试newTransactionTracer操作对后面区块交易数据重放速度的影响有多大。因此对每一百万块的数据中选取指定数据进行测试geth-query-close-debug，测试结果如下（在服务器03上进行测试）（这一部分的文档在03服务器的`/mnt/intel-disk/czm/*.log`）：

| block                          | newtransfer     | total time                                           |
| ------------------------------ | --------------- | ---------------------------------------------------- |
| 289W-290W                      | 占比大概20%-30% |                                                      |
| 389W-390W                      | 占比大概20%-30% |                                                      |
| 489W-490W                      | 占比大概20%-30% |                                                      |
| 589W-590W                      | 占比大概20%-30% |                                                      |
| 689W-690W                      | 占比大概20%-30% |                                                      |
| 689W-690W-open-debug-filter-op | 占比大概20%     | 因为取数据操作第一次取了之后就会被缓存在state DB里面 |

阶段结论：newTransactionTracer操作对合约运行的速度有一定的影响，但是每个账户的第一次数据库的读取操作是必须要进行的，因此这里没有给出优化方案。上面geth测试交易速度应该也要把账户从数据库中读取balance的操作计算在内。



### 其余服务对于geth-query的影响

**在服务器上进行测试**

首先测试注释掉除了eth之外的所有服务后，运行1720000 - 1750000区块数据的速度。

```golang

[^[[1;32mINFO ^[[0m] 2019-12-25 16:26:48 <ether-query>finish...current block = 1750001, end Block = 1750000
[^[[1;32mINFO ^[[0m] 2019-12-25 16:26:48 <ether-query>the time spent of all processBlocks 23m11.720811477s.
[^[[1;32mINFO ^[[0m] 2019-12-25 16:26:48 <ether-query>[100.00 precent]Processing block 1750001 finished, total elaspe=23m11.720818097s, lastest 1000 block elaspe=671.949µs


```

然后不去掉除eth之外的服务，运行1720000 - 1750000区块数据的速度。

```golang
[^[[1;32mINFO ^[[0m] 2019-12-25 14:50:23 <ether-query>finish...current block = 1750001, end Block = 1750000
[^[[1;32mINFO ^[[0m] 2019-12-25 14:50:23 <ether-query>the time spent of all processBlocks 23m36.077291617s.
[^[[1;32mINFO ^[[0m] 2019-12-25 14:50:23 <ether-query>[100.00 precent]Processing block 1750001 finished, total elaspe=23m36.077300161s, lastest 1000 block elaspe=24.507816ms

```

