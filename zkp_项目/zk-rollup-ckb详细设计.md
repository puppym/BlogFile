

[TOC]



# 2. 详细设计

## 2.1 背景

当前clover-protocol协议是搭建在nervous上的基于aSVC的二层协议，基于aSVC的方案主要有以下好处：

1. 用户在更新链上状态时只需要当前自己账户的proof以及整体的commit，即可更新链上的commit，不想merkletree那样需要一条proof的路径。
2. 方案相比于merkletree在生成proof以及commit以及验证过程更加轻量，不需要庞大的电路来生成证明。

缺点：

1. 每次更新commit所有用户的proof都需要更新。该commit之前的所有proof都会失效。
2. 用户需要及时知道当前自己的proof以及链上已经修改好的commit，这样会增加通讯消耗。



为了对比基于aSVC的rollup方案，本项目通过设计merkletree的rollup方案来与之进行性能对比，明确当前在rollup方案中链下状态存储具体适合于哪一种方案。



主要目标如下：

第一阶段：完成deposit，withdraw，updateAccount，transfer四条电路，通过自己构造的block.json文件进行测试，并且大致统计其证明的效率。

第二阶段：完成合约，以及链下的operator程序，并且跑通整个流程。

第三阶段：加签名验证，operaotr手续费，以及一些安全性保障的规则以及字段。

## 2.2 整体架构图



![total](C:\Users\czm18\Desktop\total.png)

**整体的流程如下：**

```
1. 用户发送onchain和offchain操作。
2. operator收集用户的offchain操作以及ethereum的onchain操作。
3. operator将链上的操作打包丢到线下的计算程序，计算状态的变更。
4. 电路根据输入来计算proof的证明。
5. operator调用submitBlock接口，查看当前证明是否能够通过。
```

## 2.3 操作的流程(流程图)



#### 2.3.1 deposit

```
1. deposit {
    to //to地址 这里对应链下状态中publicKey的公私密钥地址。
    amount
}
2. operator收集用户的offchain操作以及ethereum的onchain操作。
3. operator将链上的操作打包丢到线下的计算程序，计算状态的变更。
4. 电路根据输入来计算proof的证明。
5. operator调用submitBlock接口，查看当前证明是否能够通过。
```



#### 2.3.2 withdraw

```
1. withdraw {
    To // 链下状态中publickey对应的地址
    amount
}
2. operator收集用户的offchain操作以及ethereum的onchain操作。
3. operator将链上的操作打包丢到线下的计算程序，计算状态的变更。
4. 电路根据输入来计算proof的证明。
5. operator调用submitBlock接口，查看当前证明是否能够通过。
```



#### 2.3.3 updateAccount

```
1. updateAccount {
	own // 账户的拥有者
	publicKey
	nonce
}// offchain 操作
2. operator收集用户的offchain操作以及ethereum的onchain操作。
3. operator将链上的操作打包丢到线下的计算程序，计算状态的变更。
4. 电路根据输入来计算proof的证明。
5. operator调用submitBlock接口，查看当前证明是否能够通过。
```



#### 2.3.4 InternalTransfer

```
1. InternalTransfer {
	from
	fromAccountID
	to 
	toAccountID
	amount
	nonce
} // offchain操作
2. operator收集用户的offchain操作以及ethereum的onchain操作。
3. operator将链上的操作打包丢到线下的计算程序，计算状态的变更。
4. 电路根据输入来计算proof的证明。
5. operator调用submitBlock接口，查看当前证明是否能够通过。
```



#### 2.3.5 merkletree节点

**Account** 当前主要是merkletree一共设计为8层，账户的容量一共2^(8-1) = 2^7

```
accountID
publicKey(publicKeyX, publicKeyY)
nonce
balance
```



#### 2.3.6 链上合约设计

**链上合约的数据结构和接口**

```
data:
    operatorID // 中继的ID
    merkleRoot // 当前merkleRoot值
    map<blockNumber, blockDataHash> // 所有blockData的值
    blockNumberHeight //当前区块的高度
    pendingDeposit<address, Deposit> // 当前等待存款的数目
    pendingWithdraw<address, withdraw> // 当前等待撤款的数目

Deposit:
    // To: uint32
    // fromAccountId: uint8
    amount: uint96
    timestamp: uint64

withdraw:
    // To: uint32
    // fromAccountId : uint8
    amount: uint96
    timestamp: uint64

blockData:
    blockType: uint8
    blockNumber: uint64 // blockNumber 已经为key值
    blocksize: uint16
    // blockVersion: uint8
    merkleBefore
    merkleAfter
    data: bytes
    proof: uint256[8]

___________________________________________________________
func:
deposit() // 用户调用合约的存储接口
withdraw()
submitBlock(blockData) -> process() -> 分别处理deposit withdraw 
checkOnchainOperatorOuttime()

```

#### 2.3.7 线下状态计算程序（operator）

线下状态计算程序：

1. 监听链上的事件信息。
2. 监听用户发出的请求。
3. 使用交易池里面的交易打包块并且提交到链下程序计算提交到电路中的block.json。
4. 使用block.json文件生成proof。
5. 提交proof到链上进行验证。

该程序主要用户接收链上合约发出的onchain操作以及用户发出的offchain操作，通过将所有操作的交易汇集到交易池中，然后从交易池中取出各种交易打包成各种类型的块，然后将各种类型的区块丢到线下状态计算的程序中，计算出区块运算的中间状态，并且将中间状态作为电路的witness输入，输入到电路中进行证明的生成。

```
 XXXblockData:
     operatorID: uint16
     merkleRootBefore: String
     merkleRootAfter: String
     blockNumber: uint64
     blockType: String
     timestamp: uint64
     vector<XXXtransaction>  // XXXtransaction中的内容就是用户发出的交易数据，并且这些交易数据都会压缩后上链

listen_contract_event() // 监听合约事件信息
listen_user_tx() // 监听用户发出的事件
package_block_from_tx_pool() // 从交易池中打包交易
register_vk() // 链上注册vk
submit_block() // 提交虚拟区块
checkDepositTime() // 检查onchain的deposit操作是否超时
checkWithdrawTime() // 检查onchain的withdraw操作是否超时
withdrawWithMerkleProof() // 当交易所处于shutdown模式，提交merkleproof来进行取款

根据pool里面的各类交易数量，打包生成链下状态计算的block，然后根据该block生成带有中间数据的block.json文件 
 
    
```



#### 2.3.8 circuit

电路部分作为一个独立的程序，通过读取block.json的配置文件，然后根据不同的blockType调用不同的电路生成证明，最后将生成的证明结果输出到proof文件中。

```
setup();
depositCircuit(cs, depositBlockwitness);
withdrawCircuit(cs, withdrawBlockwitness);
updateAccountCircuit(cs, updateAccountBlockwitness);
transferAccountCircuit(cs, transferBlockwitness);
// 上面每条电路的blockSize的大小都是确定的，因为blockSize的大小一旦确定，整个电路的规模也就确定了

main() {
	pk, vk = setup()
	match block.type {
		deposit:
			depositCircuit()
		withdraw:
			withdrawCircuit()
		updateAccount:
			updateAccountCircuit()
		transfer:
			transferCircuit()
	}
	
	proof, vk
}

```

电路整体的结构如下：

![Circuit整体](C:\Users\czm18\Desktop\Circuit整体.png)

**电路分为两部份：**

1. 线下计算逻辑验证部分。
2. publicData哈希计算以及验证部分(主要验证电路中使用privateDataInput算出来的hash结果是否和电路中算出来的结果相同)。

后面介绍的电路都于updateAccountgadget电路有关，对于支付场景的逻辑，所有场景的逻辑可以抽象为A，B，operator，protocolFee账户余额的增减，因此对于updateAccountGadget的封装重用对于上述代码的编写显得尤为重要：

```json
"accountUpdateA": {
    "accountID": "", // 更新账户的accountID
    "proof": [ // merkletree的proof路径
        [],
        []
    ],
    "rootBefore": "", // 
    "rootAfter": "",
    "before": {
        "publicKeyX": "",
        "publicKeyY": "",
        "nonce": 2,
        "balances": "193236161939545530"
    },
    "After": {
        "publicKeyX": "",
        "publicKeyY": "",
        "nonce": 3,
        "balances": "193236161939545530",
    }
}
```



#### 2.3.8.1 deposit

**witness**

```
depositBlockWitness:
     operatorID: Field
     merkleRootBefore: Field
     merkleRootAfter: Filed
     blockNumber: Filed
     startIndex: Filed
     startHash: Field
     
     deposits: vector<depositwitness> 

depositWitness:
	accountMerkleRootBefore // 之前的merkleRoot
	accountUpdateA // 更新merkleRoot
	
	// publicData
	to // from的地址
	toAccountID // from在merkletree的accountID
	amount
```

**publicInput**

depositBlockData的哈希为该条电路的publicInput。

```
 DepositblockData:
     operatorID: Filed
     merkleRootBefore: Filed
     merkleRootAfter: Filed
     startHash: Filed
     endHash: Filed
     startIndex: Filed
     count: Filed
     vector<deposit.getPublicData()>  
```



#### 2.3.8.2 withdraw

**witness**

```
withdrawBlockwitness:
     operatorID: Field
     merkleRootBefore: Field
     merkleRootAfter: Filed
     blockNumber: Filed
     startIndex: Filed
     startHash: Field
     
     withdraws: vector<withdrawwitness> 

withdrawitness:
	accountMerkleRootBefor: Field
	accountupdateA
	
	// publicData
	from: Field
	fromAccountId: Field
	amount: Field
```

**publicInput**

withdrawBlockData的哈希为该条电路的publicInput。

```
 withdrawblockData:
     operatorID: Filed
     merkleRootBefore: Filed
     merkleRootAfter: Filed
     startHash: Filed
     endHash: Filed
     startIndex: Filed
     count: Filed
     vector<withdraw.getPublicData()>  
```



#### 2.3.8.3 updateAccount

**witness**

```
updateAccountBlockwitness:
     operatorID: Field
     merkleRootBefore: Field
     merkleRootAfter: Filed
     blockNumber: Filed
     startIndex: Filed
     startHash: Field
     
     updateAccounts: vector<updateAccountwitness> 

updateAccountwitness:
	accountMerkleRootBefor: Filed
	accountupdateA: Filed
	
	// publicData
	from: Filed
	fromAccountId: Filed
	publicKeyX: Filed
	publicKeyY: Filed
```

**publicInput**

updateAccountBlockData的哈希为该条电路的publicInput。

```
 updateAccountblockData:
     operatorID: Filed
     merkleRootBefore: Filed
     merkleRootAfter: Filed
     startHash: Filed
     endHash: Filed
     startIndex: Filed
     count: Filed
     vector<updateAccount.getPublicData()>  
```



#### 2.3.8.4 transfer

**witness**

```
transferBlockwitness:
     operatorID: Field
     merkleRootBefore: Field
     merkleRootAfter: Filed
     blockNumber: Filed
     startIndex: Filed
     startHash: Field
     
     transfers: vector<transferwitness> 

transferwitness:
	accountMerkleRootBefor: Filed
	accountupdateA
	accountupdateB
	
	// publicData
	from: Filed
	fromAccountId: Filed
	to: Filed
	toAccountId: Filed
	amount: Filed
```

**publicInput**

transferBlockData的哈希为该条电路的publicInput。

```
 transferblockData:
     operatorID: Filed
     merkleRootBefore: Filed
     merkleRootAfter: Filed
     startHash: Filed
     endHash: Filed
     startIndex: Filed
     count: Filed
     vector<transfer.getPublicData()>  
```



## 讨论



1. to和accountId的校验。

accountId仅仅需要在dataavailable数据中上链，因为在数据恢复阶段需要根据accountId的值去确定当前修改的叶子节点在当前merkletree中的位置。calldata里面必须要accountId，世界状态里面不需要。这里的operator可以对提交上链的block中的accountId值进行修改，用户在使用dataAvailable进行数据恢复时可以通过accountId进行简化计算过程(可以通过accountId来确定当前修改叶子节点的位置，如果没有accountId则需要便利所有的位置)，最差的情况是operator作恶，填错的accountId，导致用户在恢复dataavailable数据时accountId失效。





2. 链上交易的顺序保证。operator的顺序性保证。

链上交易顺序暂时不给予保障，链下的交易顺序由链上进行保障。



3. 链上的merkleroot如何更新。

blockData中包含的信息有：

```
 XXXblockData:
     operatorID: uint16
     merkleRootBefore: String
     merkleRootAfter: String
     blockNumber: uint64
     blockType: String
     timestamp: uint64
     vector<XXXtransaction>  // XXXtransaction中的内容就是用户发出的交易数据，并且这些交易数据
```

提交上链的虚拟区块数据为区块头和区块体，在头部包含merkleAfter的信息，blockDataHash作为publicInput，如果publicInput+proof+vk能够验证通过也就说明区块中提交的merkleAfterRoot值是正确的按照链下电路的逻辑算出来的。



## 附录

电路中block.json的输入格式：

```
{
    "operatorID": "123456",
    "blockType": 0,
    "blockNumber": 123,
    "timestamp": 1234567,
    "merkleRootBefore": "",
    "merkleRootAfter": "",
    "transactions": [
        {
            "witness": {
                "accountsMerkleRoot": "",
                "accountUpdateA": {
                    "accountID": "",
                    "proof": [
                        [],
                        []
                    ],
                    "rootBefore": "",
                    "rootAfter": "",
                    "before": {
                        "publicKeyX": "",
                        "publicKeyY": "",
                        "nonce": 2,
                        "balances": "193236161939545530"
                    },
                    "After": {
                        "publicKeyX": "",
                        "publicKeyY": "",
                        "nonce": 3,
                        "balances": "193236161939545530",
                    }
                }
            },
            "deposit": {
                "from": "",
                "fromAccountID": "",
                "amount": "",
                "txType": ""
            }
        },
        {
            "witness": {
                "accountsMerkleRoot": "",
                "accountUpdateA": {
                    "accountID": "",
                    "proof": [
                        [],
                        []
                    ],
                    "rootBefore": "",
                    "rootAfter": "",
                    "before": {
                        "publicKeyX": "",
                        "publicKeyY": "",
                        "nonce": 2,
                        "balances": "193236161939545530"
                    },
                    "After": {
                        "publicKeyX": "",
                        "publicKeyY": "",
                        "nonce": 2,
                        "balances": "193236161939545530",
                    }
                }
            },
            "accountupdate": {
                "from": "",
                "fromAccountId": "",
                "publicKeyX": "",
                "publicKeyY": "",
                "txType": ""
            }
        },
        {
            "wwitness: {
                "accountsMerkleRoot": "",
                "accountUpdateA": {
                    "accountID": "",
                    "proof": [
                        [],
                        []
                    ],
                    "rootBefore": "",
                    "rootAfter": "",
                    "before": {
                        "owner": "",
                        "publicKeyX": "",
                        "publicKeyY": "",
                        "nonce": 2,
                        "balances": "193236161939545530"
                    },
                    "After": {
                        "owner": "",
                        "publicKeyX": "",
                        "publicKeyY": "",
                        "nonce": 2,
                        "balances": "193236161939545530",
                    }
                },
                "accountUpdateB": {
                    "accountID": "",
                    "proof": [
                        [],
                        []
                    ],
                    "rootBefore": "",
                    "rootAfter": "",
                    "before": {
                        "owner": "",
                        "publicKeyX": "",
                        "publicKeyY": "",
                        "nonce": 2,
                        "balances": "193236161939545530"
                    },
                    "After": {
                        "owner": "",
                        "publicKeyX": "",
                        "publicKeyY": "",
                        "nonce": 2,
                        "balances": "193236161939545530",
                    }
                }
            },
            "transfer": {
                "from": "",
                "fromAccountID": "",
                "to": "",
                "toAccountID": "",
                "amount": 123,
                "txType": ""
            }
        },
        {
            "wiwitness {
                "accountsMerkleRoot": "",
                "accountUpdateA": {
                    "accountID": "",
                    "proof": [
                        [],
                        []
                    ],
                    "rootBefore": "",
                    "rootAfter": "",
                    "before": {
                        "owner": "",
                        "publicKeyX": "",
                        "publicKeyY": "",
                        "nonce": 2,
                        "balances": "193236161939545530"
                    },
                    "After": {
                        "owner": "",
                        "publicKeyX": "",
                        "publicKeyY": "",
                        "nonce": 2,
                        "balances": "193236161939545530",
                    }
                }
            },
            "withdraw": {
                "to": "",
                "toAccountID": "",
                "amount": 123,
                "txType": ""
            }
        }
    ]
}
```





