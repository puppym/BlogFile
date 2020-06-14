## 以太坊目录结构

```bash
accounts        	实现了一个高等级的以太坊账户管理
bmt			二进制的默克尔树的实现
build			主要是编译和构建的一些脚本和配置
cmd			命令行工具，又分了很多的命令行工具，下面一个一个介绍
	/abigen		Source code generator to convert Ethereum contract definitions into easy to use, compile-time type-safe Go packages
	/bootnode	启动一个仅仅实现网络发现的节点
	/evm		以太坊虚拟机的开发工具， 用来提供一个可配置的，受隔离的代码调试环境
	/faucet		
	/geth		以太坊命令行客户端，最重要的一个工具
	/p2psim		提供了一个工具来模拟http的API
	/puppeth	创建一个新的以太坊网络的向导
	/rlpdump 	提供了一个RLP数据的格式化输出
	/swarm		swarm网络的接入点
	/util		提供了一些公共的工具
	/wnode		这是一个简单的Whisper节点。 它可以用作独立的引导节点。此外，可以用于不同的测试和诊断目的。
common			提供了一些公共的工具类
compression		Package rle implements the run-length encoding used for Ethereum data.
consensus		提供了以太坊的一些共识算法，比如ethhash, clique(proof-of-authority)
console			console类
contracts	
core			以太坊的核心数据结构和算法(虚拟机，状态，区块链，布隆过滤器)
crypto			加密和hash算法，
eth			实现了以太坊的协议
ethclient		提供了以太坊的RPC客户端
ethdb			eth的数据库(包括实际使用的leveldb和供测试使用的内存数据库)
ethstats		提供网络状态的报告
event			处理实时的事件
les			实现了以太坊的轻量级协议子集
light			实现为以太坊轻量级客户端提供按需检索的功能
log			提供对人机都友好的日志信息
metrics			提供磁盘计数器
miner			提供以太坊的区块创建和挖矿
mobile			移动端使用的一些warpper
node			以太坊的多种类型的节点
p2p			以太坊p2p网络协议
rlp			以太坊序列化处理
rpc			远程方法调用
swarm			swarm网络处理
tests			测试
trie			以太坊重要的数据结构Package trie implements Merkle Patricia Tries.
whisper			提供了whisper节点的协议。
```

## ethereum全节点工作流程

### ethereum 全节点服务启动过程

1. 入口：go-ethereum -> cmd -> geth -> main

2. Initialize the CLI app and start Geth:

   1. 初始化 `*cli.App`：`init()`
   2. 启动 Geth 节点（geth is the main entry point into the system）: `geth(ctx *cli.Context)`

3. 创建全节点: `makeFullNode(ctx *cli.Context)`

   1. 创建节点： `makeConfigNode(ctx *cli.Context)`

      - 加载配置：loadConfig
      - SetNodeConfig: SetNodeConfig applies node-related command line flags to the config.

      `SetNodeConfig func(ctx *cli.Context, cfg *node.Config)`

      - creates a new P2P node: `New(conf *Config) (*Node, error)`
      - set Eth config: applies eth-related command line flags to the config.

      `SetEthConfig func(ctx *cli.Context, stack *node.Node, cfg *eth.Config)`

      - SetShhConfig: `SetShhConfig func(ctx *cli.Context, stack *node.Node, cfg *whisper.Config)`
      - SetDashboardConfig: `SetDashboardConfig func(ctx *cli.Context, cfg *dashboard.Config)`

   2. 注册 ETH service：`RegisterEthService(stack *node.Node, cfg *eth.Config)`:

      - node 注册 service：`Register(constructor ServiceConstructor)`

      append serviceFuncs

      - service: New creates a new Ethereum object `New(ctx *node.ServiceContext, config *Config) (*Ethereum, error)`

   3. 注册 Dashboard service: `RegisterDashboardService(stack *node.Node, cfg *dashboard.Config, commit string)`

   4. 注册 shh service: `RegisterShhService(stack *node.Node, cfg *whisper.Config)`

   5. 注册 GraphQL service: `RegisterGraphQLService(stack *node.Node, endpoint string, cors, vhosts []string, timeouts rpc.HTTPTimeouts)`

   6. 注册 EthStats service: `RegisterEthStatsService(stack *node.Node, url string)`

4. 启动节点：`startNode(ctx *cli.Context, stack *node.Node)`

   1. 启动节点：`StartNode(stack *node.Node)`
      - execute serviceFuncs: `service, err := constructor(ctx)`
      - append service Protocols
      - server start: `func (srv *Server) Start() (err error)`
      - service start: `Start(server *p2p.Server)`
      - start RPC: `startRPC func(services map[reflect.Type]Service) error`
   2. unlock Accounts: `unlockAccounts func(ctx *cli.Context, stack *node.Node)`
   3. 注册 wallet：
   4. 创建 client：
   5. 设置 rpc client: `SetContractBackend func(backend bind.ContractBackend)`

   sets a rpc client which connecting to our local node.

   1. 订阅 Event Mux:
   2. 启动挖矿

### ethereum eth service执行过程

一个node包含很多service，eth service是以太坊服务主要用于同步区块和执行区块中的交易，执行交易过程使用core包下面的库文件，它实现于Service接口。eth service通过node的Register方法注入。以下是eth包下子包：

- downloader 主要用于和网络同步，包含了传统同步方式和快速同步方式
- fetcher 主要用于基于块通知的同步，接收到当我们接收到NewBlockHashesMsg消息得时候，我们只收到了很多Block的hash值。 需要通过hash值来同步区块。
- filter 提供基于RPC的过滤功能，包括实时数据的同步(PendingTx)，和历史的日志查询(Log filter)
- gasprice 提供gas的价格建议， 根据过去几个区块的gasprice，来得到当前的gasprice的建议价格

eth service启动之后其主要执行过程如下：

1. Ethereum服务创建之后会调用其Start方法，`func (s *Ethereum) Start(srvr *p2p.Server)`，依次启动以下服务：1. 启动布隆过滤器请求处理的goroutine。2. 启动RPC服务，3. 启动协议管理器`s.protocolManager.Start(maxPeers)`，该服务管理着以太坊的子协议。

2. Ethereum启动协议管理器`(pm *ProtocolManager) Start(maxPeers int)`。分别循环广播交易和区块，syncer负责周期性地与网络同步，下载散列和块以及处理通知处理程序。，txsyncLoop负责每个新连接的初始事务同步。 当新的对等体出现时，我们转发所有当前待处理的事务。 为了最小化出口带宽使用，我们一次将一个小包中的事务发送给一个对等体。

3. pm.syncer() -》 pm.synchronise(pm.peers.BestPeer()尝试让本地节点和远程节点同步 -》  pm.downloader.Synchronise(peer.id, pHead, pTd, mode) -》 d.synchronise(id, head, td, mode)  -》  d.syncWithPeer(p, hash, td)从一个明确节点的某一hash头部开始同步，启动几个fetcher 分别负责header,bodies,receipts,处理headers，并且根据不同同步方式增加新的处理逻辑  -》spawnSync给每个fetcher启动一个goroutine, 然后阻塞的等待fetcher出错，然后就是分别执行上面的fetch逻辑。

4. fetchHeaders方法用来获取header。 然后根据获取的header去获取body和receipt等信息。fetchBodies函数定义了一些闭包函数，然后调用了fetchParts函数。receipt的处理和body类似。processHeaders方法，这个方法从headerProcCh通道来获取header。并把获取到的header丢入到queue来进行调度，这样body fetcher或者是receipt fetcher就可以领取到fetch任务。

5. 然后FastSync执行processFastSyncContent函数，FullSync执行processFullSyncContent函数。按照不同的同步方式在链上插入状态。

6. -》processFullSyncContent -》d.importBlockResults(results) -》d.blockchain.InsertChain(blocks)向本地链中成批插入区块  -》 bc.insertChain(chain, true) InsertChain的内部实现 -》 bc.processor.Process(block, statedb, bc.vmConfig) 循环处理每一个区块 -》core/state_processor.go ApplyTransaction(p.config, p.bc, nil, gp, statedb, header, tx, usedGas, cfg)执行每一笔交易。

7. ApplyMessage(vmenv, msg, gp)在当前状态的基础上执行这笔交易 -》 NewStateTransition(evm, msg, gp).TransitionDb() -》evm.Create(sender, st.data, st.gas, st.value) 或者evm.Call(sender, st.to(), st.data, st.gas, st.value)按照字节码执行交易。

8. evm.Call(sender, st.to(), st.data, st.gas, st.value) -》run(evm, contract, input, false)运行合约字节码 -》interpreter.Run(contract, input, readOnly) 执行每一层智能合于字节码 -》operation.execute(&pc, in, contract, mem, stack)执行每个指令对应的函数。

    

### state DB缓存

[stateDB底层设计]([http://lessisbetter.site/2018/06/22/ethereum-code-statedb/#%E5%BA%95%E5%B1%82%E5%AD%98%E5%82%A8%E8%AE%BE%E8%AE%A1](http://lessisbetter.site/2018/06/22/ethereum-code-statedb/#底层存储设计))

### 关键结构体

eth/backend.go中包含Ethereum结构体的定义和其成员函数，结构体如下：

```golang
type Service interface {
	// Protocols retrieves the P2P protocols the service wishes to start.
	Protocols() []p2p.Protocol

	// APIs retrieves the list of RPC descriptors the service provides
	APIs() []rpc.API

	// Start is called after all services have been constructed and the networking
	// layer was also initialized to spawn any goroutines required by the service.
	Start(server *p2p.Server) error

	// Stop terminates all goroutines belonging to the service, blocking until they
	// are all terminated.
	Stop() error
}

// Ethereum结构体是对上面接口的实现
// Ethereum implements the Ethereum full node service.
type Ethereum struct {
	config      *Config					配置
	chainConfig *params.ChainConfig		链配置

	// Channel for shutting down the service
	shutdownChan  chan bool    // Channel for shutting down the ethereum
	stopDbUpgrade func() error // stop chain db sequential key upgrade

	// Handlers
	txPool          *core.TxPool			交易池
	blockchain      *core.BlockChain		区块链
	protocolManager *ProtocolManager		协议管理
	lesServer       LesServer				轻量级客户端服务器

	// DB interfaces
	chainDb ethdb.Database // Block chain database	区块链数据库

	eventMux       *event.TypeMux
	engine         consensus.Engine				一致性引擎。 应该是Pow部分
	accountManager *accounts.Manager			账号管理

	bloomRequests chan chan *bloombits.Retrieval // Channel receiving bloom data retrieval requests	接收bloom过滤器数据请求的通道
	bloomIndexer  *core.ChainIndexer             // Bloom indexer operating during block imports  //在区块import的时候执行Bloom indexer操作 暂时不清楚是什么

	ApiBackend *EthApiBackend		//提供给RPC服务使用的API后端

	miner     *miner.Miner			//矿工
	gasPrice  *big.Int				//节点接收的gasPrice的最小值。 比这个值更小的交易会被本节点拒绝
	etherbase common.Address		//矿工地址

	networkId     uint64			//网络ID  testnet是0 mainnet是1 
	netRPCService *ethapi.PublicNetAPI	//RPC的服务

	lock sync.RWMutex // Protects the variadic fields (e.g. gas price and etherbase)
}

type BlockChain struct {
	config *params.ChainConfig // chain & network configuration

	hc            *HeaderChain		// 只包含了区块头的区块链
	chainDb       ethdb.Database	// 底层数据库
	rmLogsFeed    event.Feed		// 下面是很多消息通知的组件
	chainFeed     event.Feed
	chainSideFeed event.Feed
	chainHeadFeed event.Feed
	logsFeed      event.Feed
	scope         event.SubscriptionScope
	genesisBlock  *types.Block		// 创世区块

	mu      sync.RWMutex // global mutex for locking chain operations
	chainmu sync.RWMutex // blockchain insertion lock
	procmu  sync.RWMutex // block processor lock

	checkpoint       int          // checkpoint counts towards the new checkpoint
	currentBlock     *types.Block // Current head of the block chain 当前的区块头
	currentFastBlock *types.Block // Current head of the fast-sync chain (may be above the block chain!) 当前的快速同步的区块头.

	stateCache   state.Database // State database to reuse between imports (contains state cache)
	bodyCache    *lru.Cache     // Cache for the most recent block bodies
	bodyRLPCache *lru.Cache     // Cache for the most recent block bodies in RLP encoded format
	blockCache   *lru.Cache     // Cache for the most recent entire blocks
	futureBlocks *lru.Cache     // future blocks are blocks added for later processing  暂时还不能插入的区块存放位置.

	quit    chan struct{} // blockchain quit channel
	running int32         // running must be called atomically
	// procInterrupt must be atomically called
	procInterrupt int32          // interrupt signaler for block processing
	wg            sync.WaitGroup // chain processing wait group for shutting down

	engine    consensus.Engine	// 一致性引擎
	processor Processor // block processor interface  // 区块处理器接口
	validator Validator // block and state validator interface // 区块和状态验证器接口
	vmConfig  vm.Config //虚拟机的配置

	badBlocks *lru.Cache // Bad block cache  错误区块的缓存.
}
```





## 参考文档

* [ethereum源码分析](https://github.com/ZtesoftCS/go-ethereum-code-analysis)
* [stateDB存储设计](http://lessisbetter.site/2018/06/22/ethereum-code-statedb/#底层存储设计)