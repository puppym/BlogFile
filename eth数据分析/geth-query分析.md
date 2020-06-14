### geth-query  分析

主要的函数为`func traceBlock(ethereum *eth.Ethereum, block *types.Block) (*traceData, error)`

，通过该函数来处理每一个block的状态。

`data := &traceData{}`

```golang
type traceData struct {
	transactions []*transactionTrace
	transfers    []*valueTransfer
	events       []*types.Log
	traces       []*debugTrace
}
```

通过`statedb, err := blockchain.StateAt(blockchain.GetBlockByHash(block.ParentHash()).Root())`获取到每个block的状态，然后基于当前状态通过执行block中的transaction来抽取执行每一个transaction时的数据。

#### blocks

直接获取到的区块数据的顺序改变顺序后存入blocks表中。

#### transfers表

关于transfer表，在处理block的时候一共有四处地方应当添加transfer表的内容：

1. `recordRewards()`主要时通过计算叔叔区块和父亲区块的奖励，来填充data中的transfer表。

2. `tracer := newTransactionTracer(statedb, tx, block.Header(), config)`每一个transaction操作，如何不是`CREATE`，那么就是一个`TRANSACTION`操作，因为直接为后者操作可能会直接就是转账，因此最外层需要进行value值的判断。address.transfer()函数再EVM中会变成call指令。

3. `filterTraces`之后过滤出`CREATE，CALL，CALLCODE，DELEGATECALL，SELFDESTRUCT`5条指令，然后基于这5条指令来计算transfer。

4. 每个区块中`transactions`的gas消耗也会组成一个transfer交易发送给矿工。

   ```golang
   data.transfers = append(data.transfers, newTransfer(statedb, 0, tx.Hash(), from,
   			block.Coinbase(), new(big.Int).Mul(new(big.Int).SetUint64(trace.receipt.GasUsed), tx.GasPrice()), "FEE"))
   ```

   

#### traces

在以太坊中一个transaction能够触发很多原子的trace一些区块链浏览器也把它称为内部交易。

`traces`表的数据主要是在`filterTraces`函数中过滤出`CREATE，CALL，CALLCODE，DELEGATECALL，SELFDESTRUCT`5个指令以及调用预编译合约的trace动作过滤出来，然后存入traces表中。

By using the three main trace types (call, create, suicide), we can generalise the type of ether transfers that can occur:

- **call**: Used to **transfer** ether from one account to another and/or to call a smart contract function defined by parameters in the data field. This trace also encompasses **delegatecall** and **callcode**.
- **create**: Used to create a smart contract, and ether is **transferred** to the newly created smart contract
- **suicide**: Used by an owner of a smart contract to kill the smart contract. Triggers a **transfer** of ether for a refund for killing a contract. Additionally, killing a smart contract can free memory in the blockchain, which can also affects the value transferred.
- [ethereum-traces-not-transactions-3f0533d26aa](https://tnie.github.io/2016/11/30/statically-bound-And-dynamically-bound/)

#### events

通过`trace.receipt, err = traceTx(config, blockchain, nil,gasPool, statedb, block.Header(), tx, usedGas, vmConfig)`获取每一个`transaction`的`trace.recepit`，然后从`trace.receipt.Logs`直接取出`events`。这个表没有进额外的计算。

```golang
type transactionTrace struct {
	receipt *types.Receipt
	// logs    []*vm.Log
	err error
}


type Receipt struct {
	// Consensus fields: These fields are defined by the Yellow Paper
	PostState         []byte `json:"root"`
	Status            uint64 `json:"status"`
	CumulativeGasUsed uint64 `json:"cumulativeGasUsed" gencodec:"required"`
	Bloom             Bloom  `json:"logsBloom"         gencodec:"required"`
	Logs              []*Log `json:"logs"              gencodec:"required"`

	// Implementation fields: These fields are added by geth when processing a transaction.
	// They are stored in the chain database.
	TxHash          common.Hash    `json:"transactionHash" gencodec:"required"`
	ContractAddress common.Address `json:"contractAddress"`
	GasUsed         uint64         `json:"gasUsed" gencodec:"required"`

	// Inclusion information: These fields provide information about the inclusion of the
	// transaction corresponding to this receipt.
	BlockHash        common.Hash `json:"blockHash,omitempty"`
	BlockNumber      *big.Int    `json:"blockNumber,omitempty"`
	TransactionIndex uint        `json:"transactionIndex"`
}

// 注意这里的trace是transactionTrace
if len(trace.receipt.Logs) > 0 {
			data.events = append(data.events, trace.receipt.Logs...)
		}
```

#### transactions

主要是通过代码`tracer := newTransactionTracer(statedb, tx, block.Header(), config)`抽离出transactions的数据结构，然后通过代码`data.transactions = append(data.transactions, trace)`添加到transactions表中。

#### 预编译合约

trace表追踪的预编译合约有`CREATE，CALL，CALLCODE，DELEGATECALL，SELFDESTRUCT`。

`CREATE,CALL`能够导致transfers 到一个账户。

`CALLCODE,DELEGATECALL`，虽然不能直接进行转账，但是能够创造一个能够跳转到一个能够转账的函数。

`SELFDESTRUCT`能够销毁合约，给销毁账户返回一定的gas费用，所以能够导致转账操作。

对于预编译合约而言一共有以下的合约代码，通过跟踪预编译合约的trace，可以用来分析一些合约对预编译合约的调用情况。

```golang
var PrecompiledContractsIstanbul = map[common.Address]PrecompiledContract{
	common.BytesToAddress([]byte{1}): &ecrecover{},
	common.BytesToAddress([]byte{2}): &sha256hash{},
	common.BytesToAddress([]byte{3}): &ripemd160hash{},
	common.BytesToAddress([]byte{4}): &dataCopy{},
	common.BytesToAddress([]byte{5}): &bigModExp{},
	common.BytesToAddress([]byte{6}): &bn256AddIstanbul{},
	common.BytesToAddress([]byte{7}): &bn256ScalarMulIstanbul{},
	common.BytesToAddress([]byte{8}): &bn256PairingIstanbul{},
	common.BytesToAddress([]byte{9}): &blake2F{},
}
```



### geth-query 性能分析

目前由于遇到geth-query同步的性能问题，特此对geth-query进行性能分析。

关于`EVMInterpreter`的配置在开启全节点的时候是debug模式，因此要额外捕获一起traces进行记录，在下面记录debug的模块：

1. `interpreter.go 182行`

```golang
if in.cfg.Debug {
		defer func() {
			if err != nil {
				if !logged {
					in.cfg.Tracer.CaptureState(in.evm, pcCopy, op, gasCopy, cost, mem, stack, contract, in.evm.depth, err)
				} else {
					in.cfg.Tracer.CaptureFault(in.evm, pcCopy, op, gasCopy, cost, mem, stack, contract, in.evm.depth, err)
				}
			}
		}()
	}
```

2. `interpreter.go 198行`

```golang
if in.cfg.Debug {
			// Capture pre-execution values for tracing.
			logged, pcCopy, gasCopy = false, pc, contract.Gas
		}
```

3. `interpreter.go 264`

```golang
if in.cfg.Debug {
			in.cfg.Tracer.CaptureState(in.evm, pc, op, gasCopy, cost, mem, stack, contract, in.evm.depth, err)
			logged = true
		}
```

4. `logger.go 197`

```golang
if l.cfg.Debug {
		fmt.Printf("0x%x\n", output)
		if err != nil {
			fmt.Printf(" error: %v\n", err)
		}
	}
```







#### bug 清除

1. revert指令没有过滤，导致stack和op.depth不匹配。

2. 下面这个交易在depth为1的层revert之后还有指令。

block：6899904

tx:        0xca3a9eb8338aaaeecb1981e605c5bb52b057ff04c60e21803fae72bde5952bd1

```golang
vmConfig := vm.Config{
			Debug:  true,
			Tracer: vm.NewStructLogger(logconfig),
}
vmConfig 写在交易循环的外面，导致每个块中每个交易的trace是增量增加的形式，导致内存使用量过大。		
```

3. 交易call之后depth一次增加了两层。

原因是staticcode指令没有统计进来

block: 6899956

Tx: 0xd9baa7020e4c18434807391e3ee7c8df8346b1c39908bd53fbc6a9677dc61d5d

4. 如果交易失败直接通过交易的状态识别，不必从trace当中查找transfer交易。

5. 处理suicide和selfdestruct指令的回退费用。

suicide和selfdestruct指令都要进行在当前栈中添加transfer操作。因为以前合约使用的指令为suicide和当前为selfdestruct，因此两者应该都要进行考虑。

block: 56379

Tx: 0xfa687c11c689b704f1109358a2abe3bf5545b450591183d3c890ed87a110fb2f

当前处理是将selfDestruct指令中合约账户的值通过err返回出来，然后再AddStructLog中进行处理。处理完后将err值回执成nil。

6. 执行失败的call指令。

0xd8c6964481777d2eb5c3ad2062235347ebd87855df6fc6e78807c305636033e8

197448

这个交易是一个withdraw交易，相当于对于call，想去执行转账操作，但是没有成功，我们还是计算了transfer。因此对于call指令应该通过查看err查看是否执行成功。

6. 从代码中查看call和DelegateCall

7. 进入CaptureState 中err不为NULL的情况要就是outofgas的情况，需要单独进行处理。outofgas是一种错误，opcode 没有正常执行。cantransfer是一种正常情况，指令在正常执行，尽管执行报错，这种报错后面还会继续执行指令。

   

| depth | len(stack) | opcode |
| ----- | ---------- | ------ |
| 1     | 1          | call   |
| 1     | 2          | swap   |
|       | 1          | ...    |

8. 对于contrace结构体而言

```golang
// Contract represents an ethereum contract in the state database. It contains
// the contract code, calling arguments. Contract implements ContractRef
type Contract struct {
	// CallerAddress is the result of the caller which initialised this
	// contract. However when the "call method" is delegated this value
	// needs to be initialised to that of the caller's caller.
	// 合约的拥有者
	CallerAddress common.Address
	// 调用合约的地址
	caller        ContractRef
	// 合约自身的地址
	self          ContractRef

	jumpdests map[common.Hash]bitvec // Aggregated result of JUMPDEST analysis.
	analysis  bitvec                 // Locally cached result of JUMPDEST analysis

	Code     []byte
	CodeHash common.Hash
	CodeAddr *common.Address
	Input    []byte

	Gas   uint64
	value *big.Int
}
```

9. 异常交易

[1722681]trace transaction...txid=0xe0298b38dd2ff455c84585bb5acf0d6342aec147e87bf6b409f2fc5b86e082b8

原因： call 指令在某一层  out of gas 只会将当层的stack抛掉。 （这个合约貌似是攻击合约）



[1703914]0xb6091f2a7712dde5af6d77a41830d685d44e957e568c04cbf3d2e4ddcd7e2bfe

该交易栈的大小和其深度不匹配，栈的大小没有进行退栈。

**未过滤trace**

```golang
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: PUSH2.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMPDEST.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: SWAP1.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMPDEST.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: ISZERO.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: PUSH2.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMPI.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMPDEST.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: PUSH2.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 3, len(self.stack): 4, opcode: SWAP4.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 3, len(self.stack): 3, opcode: POP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:17:43 <ether-query>entry.Depth: 3, len(self.stack): 3, opcode: POP.

```



**过滤trace**

```golang
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 1, len(self.stack): 1, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 1, len(self.stack): 2, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 1, len(self.stack): 2, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 2, len(self.stack): 2, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 2, len(self.stack): 3, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 2, len(self.stack): 3, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 3, len(self.stack): 3, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 3, len(self.stack): 3, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 3, len(self.stack): 3, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 3, len(self.stack): 4, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: RETURN.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 3, len(self.stack): 4, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: RETURN.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 3, len(self.stack): 4, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: RETURN.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 3, len(self.stack): 4, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 3, len(self.stack): 4, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 5, len(self.stack): 5, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 6, len(self.stack): 6, opcode: RETURN.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 5, len(self.stack): 6, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 5, len(self.stack): 5, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 5, len(self.stack): 5, opcode: RETURN.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 4, len(self.stack): 5, opcode: CALL.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 5, len(self.stack): 5, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 5, len(self.stack): 5, opcode: RETURN.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 4, len(self.stack): 5, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 4, len(self.stack): 4, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 3, len(self.stack): 4, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 3, len(self.stack): 3, opcode: RETURN.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 2, len(self.stack): 3, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 2, len(self.stack): 2, opcode: RETURN.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 1, len(self.stack): 2, opcode: JUMP.
[^[[1;32mINFO ^[[0m] 2019-12-25 12:5:15 <ether-query>entry.Depth: 1, len(self.stack): 1, opcode: RETURN.


```

该问题出现的原因是JUMP可能会导致opcode Depth的变化，因此需要过滤JUMP

