### transfer 抽取

1. `recordRewards()`主要时通过计算叔叔区块和父亲区块的奖励，来填充data中的transfer表。

```golang
type StateTransition struct {
    // 区块内部gas使用数量
	gp         *GasPool
	msg        Message
	// 当前gas的余量
	gas        uint64
	gasPrice   *big.Int
	initialGas uint64
	value      *big.Int
	data       []byte
	state      vm.StateDB
	evm        *vm.EVM
}
```

1. `tracer := newTransactionTracer(statedb, tx, block.Header(), config)`每一个transaction操作，如何不是`CREATE`，那么就是一个`TRANSACTION`操作，因为直接为后者操作可能会直接就是转账，因此最外层需要进行value值的判断。address.transfer()函数再EVM中会变成call指令。
2. `filterTraces`之后过滤出`CREATE，CALL，CALLCODE，DELEGATECALL，SELFDESTRUCT`5条指令，然后基于这5条指令来计算transfer。
3. 每个区块中`transactions`的gas消耗也会组成一个transfer交易发送给矿工。



对于transfer值的抽取一共再一下几个地方进行数据抽取：

1. state_transition.go 232  每一笔交易的gas费用 (st.initialGas - st.gas)

```golang
	if st.evm.Interpreter().(*vm.EVMInterpreter).GetConfig().Debug {
		st.evm.Interpreter().(*vm.EVMInterpreter).GetConfig().Tracer.(*vm.StructLogger).CaptureTransferState(st.evm.StateDB.(*state.StateDB), 0, common.Hash{}, msg.From(),
			st.evm.Coinbase, new(big.Int).Mul(new(big.Int).SetUint64(st.gasUsed()), st.gasPrice), "GASFEE", 0)
	}
```

2. evm.go   420                    创建合约的转账值

```golang
	if evm.Interpreter().(*EVMInterpreter).GetConfig().Debug {
		evm.Interpreter().(*EVMInterpreter).GetConfig().Tracer.(*StructLogger).CaptureTransferState(evm.StateDB.(*state.StateDB), evm.depth, common.Hash{}, caller.Address(),
			address, value, "CREATE", evm.StateDB.(*state.StateDB).GetNextRevisionId())
	}
```

3.  evm.go 240       call 指令的转账

```golang
	if evm.Interpreter().(*EVMInterpreter).GetConfig().Debug {
		evm.Interpreter().(*EVMInterpreter).GetConfig().Tracer.(*StructLogger).CaptureTransferState(evm.StateDB.(*state.StateDB), evm.depth, common.Hash{}, caller.Address(),
			to.Address(), value, "CALL", evm.StateDB.(*state.StateDB).GetNextRevisionId())
	}
```

3. instructions.go   881      selfdestruct返回的合约账户balance

```golang
	if interpreter.GetConfig().Debug {
		interpreter.GetConfig().Tracer.(*StructLogger).CaptureTransferState(interpreter.evm.StateDB.(*state.StateDB), interpreter.evm.depth, common.Hash{}, contract.Address(),
			common.BigToAddress(stack.pop()), balance, "selfDestruct", interpreter.evm.StateDB.(*state.StateDB).GetNextRevisionId())
	}
```
