### 3.2 blocks

| Key              |                                    |                          |
| ---------------- | ---------------------------------- | ------------------------ |
| blockNumber      | uint64                             | bigint                   |
| blockHash        | a hex string                       | text (32 byte Keccak256) |
| parentHash       | a hex string                       | text (32 byte Keccak256) |
| nonce            | uint64                             | bigint                   |
| miner            | a hex string                       | text (Address length 20) |
| difficulty       | bigint_to_string                   | text                     |
| totalDifficulty  | bigint_to string                   | bigint                   |
| extraData        | byte[]                             | text                     |
| size             | float64->string                    | text                     |
| gasLimit         | uint64                             | bigint                   |
| gasUsed          | uint64                             | bigint                   |
| _timestamp       | uint64                             | bigint                   |
| transactionCount | string  transactionDataLen->string | bigint                   |

### 3.3 transactions

| key             |                          |                          |
| --------------- | ------------------------ | ------------------------ |
| id              | uint64  uint64 -> string | text (32 byte Keccak256) |
| blockNumber     | uint64                   | bigint                   |
| blockHash       | a hex string             | text (32 byte Keccak256) |
| _timestamp      | uint64                   | bigint                   |
| transactionHash | a hex string             | text (32 byte Keccak256) |
| _from           | a hex string             | text                     |
| _to             | a hex string             | text                     |
| gas             | uint64                   | bigint                   |
| gasUsed         | uint64                   | bigint                   |
| gasPrice        | big.Int                  | bigint                   |
| input           | []byte                   | text                     |
| logs            | log                      | text                     |
| nonce           | uint64                   | bigint                   |
| txStr           | string                   | text                     |
| contractAddress | string                   | text                     |
| error           | string                   | text                     |
| decollator      | ""                       | text                     |

### 3.4 transfers

| key             |                          |        |
| --------------- | ------------------------ | ------ |
| id              | uint64  uint64 -> string | bigint |
| blockNumber     | uint64                   | bigint |
| blockHash       | a hex string             | text   |
| _timestamp      | uint64                   | bigint |
| transactionHash | a hex string             | text   |
| transferIndex   | int                      | bigint |
| depth           | int                      | bigint |
| _from           | a hex string             | text   |
| _to             | a hex string             | text   |
| fromBalance     | string                   | text   |
| toBalance       | string                   | text   |
| transferValue   | string                   | bigint |
| transferType    | string                   | text   |
| decollator      | ""   ->  string          | text   |

### 3.5 traces

`trace.go 96`



trace包含内容为：CREATE，CALL，CALLCODE，DELEGATECALL，SELFDESTRUCT操作及预编译合约的调用。

| key           |                  |                          |
| ------------- | ---------------- | ------------------------ |
| id            | uint64           | bigint                   |
| blockHash     | [HashLength]byte | text (32 byte Keccak256) |
| blockHeight   | uint64           | bigint                   |
| txHash        | [HashLength]byte | text (32 byte Keccak256) |
| txIndex       | int              | bigint                   |
| pc            | int              | bigint                   |
| op            | string           | string                   |
| depth         | int              | bignint                  |
| refundCounter | int              | bigint                   |
| precompiled   | string           | text                     |
| err           | string           | text                     |
| decollator    | ""               | text                     |

### 3.6 events



| key              |                                    |         |
| ---------------- | ---------------------------------- | ------- |
| id               | uint64+hash                        | bigint  |
| eventAddress     | [AddressLength]byte   a hex string | text    |
| topics           | []byte -> string                   | text    |
| eventData        | []byte -> string                   | text    |
| blockNumber      | uint64                             | bigint  |
| transactionHash  | [HashLength]byte                   | text    |
| transactionIndex | uint                               | bignint |
| blockHash        | [HashLength]byte                   | text    |
| logIndex         | uint                               | bigint  |
| removed          | bool                               | boolean |
| decollator       | ""                                 | text    |





### 问题指南

1. `blocks`代码中`gasLimit`和`getUsed`混淆。
2. `events`当中`blocksHash`有问题。
3. `traces`中新加了一行ID。
4. 分割行中bigint空白的行值应该为0，不能为""。
5. `traces`分割行逗号多四行。









