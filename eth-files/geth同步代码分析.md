api_tracer.go文件

```golang
// PrivateDebugAPI is the collection of Ethereum full node APIs exposed over
// the private debugging endpoint.
type PrivateDebugAPI struct {
	eth *Ethereum
}
```

该文件有一些traceblock，tracetx操作。

这里的traceBlock操作没有看明白，这个函数可能有助于理解geth-query中traceBlock函数的优化。



state_precoessor.go

```golang
// StateProcessor is a basic Processor, which takes care of transitioning
// state from one point to another.
//
// StateProcessor implements Processor.
type StateProcessor struct {
	config *params.ChainConfig // Chain configuration options
	bc     *BlockChain         // Canonical block chain
	engine consensus.Engine    // Consensus engine used for block rewards
}

// Process processes the state changes according to the Ethereum rules by running
// the transaction messages using the statedb and applying any rewards to both
// the processor (coinbase) and any included uncles.
//
// Process returns the receipts and logs accumulated during the process and
// returns the amount of gas that was used in the process. If any of the
// transactions failed to execute due to insufficient gas it will return an error.
func (p *StateProcessor) Process(block *types.Block, statedb *state.StateDB, cfg vm.Config) (types.Receipts, []*types.Log, uint64, error)
```

能够通过世界状态和区块信息直接执行一个区块。