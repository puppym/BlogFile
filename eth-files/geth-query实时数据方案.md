## geth-query实时数据自动导出和导入

为了获得实时重放数据，我们需要一边运行geth同步区块，一边使用geth-query重放同步的区块获取重放的中间数据。暂时想到了一下解决方案：

### 总体方案

1. go-ethereum程序和geth-query程序分开放置，但是这样存在一个问题，level DB数据库设计不运行两个实例同时进行访问，导致两个分开的程序不能同时访问levelDB数据库。
2. geth-query程序作为一个服务注册在geth的服务中，两者共用一个level DB实例。在启动geth程序的时候可以指定startBlock和endBlock来并行快速执行这段区块，执行完成后，会校验endBlock+1是否小于同步到的最新区块，如果小于geth-query工具就单线程重放区块，导出区块交易重放数据的csv文件。

### 方案代码

针对方案2的代码一共有以下：

1. geth-query工具代码，该代码主要主要是定点并行重放执行区块。[gethQuery](https://aciclo.net/zimin.chen/gethQuery)
2. 带geth-query服务的go-ethereum代码，该代码主要用于重放区块和同步区块同时执行。[go-ethereum](https://aciclo.net/zimin.chen/go-ethereum)
3. 不带geth-query服务的go-ethereum代码，该代码主要用于同步区块。[go-ethereum](https://github.com/ethereum/go-ethereum)

### 执行步骤

针对方案2，以下是整套方案的执行步骤：

1. 首先使用不带geth-query的go-ethereum代码同步区块。同步到最新后终止同步。
2. 使用geth-query工具重放已经同步的区块，导出重放中间数据成csv文件。注意为了使得相邻的1000个区块不分成两个csv文件，因此startBlock的值为1或者1000的整数倍，endBlock的值为1000*N+999。
3. 使用`import_check_createIndex.py`脚本将geth-query的数据导入到postgres数据库。
4. 启动带geth-query服务的go-ethereum代码，该代码也能够并行重放一段区块，然后单线程重放区块。startBlock的值要和上面endBlock的值能够连接上，startBlock为1000\*(N+1)，endBlock的值为1000*M+999。
5. geth-query服务并行重放区块完成后开始单线程重放区块时，执行`import_data_each_1000.py`脚本，至此，该工具能够自动同步数据和导出数据，以及将导出数据导入到csv文件中。
6. 终止上述程序的步骤首先终止`import_data_each_1000.py`脚本，然后终止带geth-query服务的geth程序，然后根据最后一行的日志删除最后1000个区块的6个csv文件，因为该csv文件没有完全同步下面这1000个区块，下次同步不好确定启动位置。

### 注意事项

1. 步骤2 中startBlock的值为1或者1000的整数倍，endBlock的值为1000*N+999。
2. 步骤4中startBlock的值要和上面endBlock的值能够连接上，startBlock为1000\*(N+1)，endBlock的值为1000*M+999。
3. 步骤六中删除最后1000个区块的6个csv文件，因为该csv文件没有完全同步下面这1000个区块，下次同步不好确定启动位置。