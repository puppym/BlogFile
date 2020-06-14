**blocks**

| Key              | evmType         | pgType        |
| ---------------- | --------------- | ------------- |
| blockNumber      | bigint          | bigint        |
| blockHash        | hex hash        | text          |
| parentHash       | hex hash        | text          |
| nonce            | [8]byte(uint64) | numeric(20,0) |
| miner            | address hash    | text          |
| difficulty       | bigint          | numeric(32,0) |
| totalDifficulty  | bigint          | numeric(32,0) |
| extraData        | byte[]          | text          |
| size             | string format   | text          |
| gasLimit         | uint64          | bigint        |
| gasUsed          | uint64          | bigint        |
| _timestamp       | uint64          | bigint        |
| transactionCount | uint64          | int           |

**transfers**

| key             | evmType         | pgType        |
| --------------- | --------------- | ------------- |
| id              | bigint          | bigint        |
| blockNumber     | bigint          | bigint        |
| blockHash       | hex hash        | text          |
| _timestamp      | uint64          | bigint        |
| transactionHash | hex hash        | text          |
| transferIndex   | int             | integer       |
| depth           | int             | integer       |
| _from           | address hash    | text          |
| _to             | address hash    | text          |
| fromBalance     | bigint          | numeric(78,0) |
| toBalance       | bigint          | numeric(78,0) |
| transferValue   | bigint          | numeric(78,0) |
| transferType    | string          | text          |
| decollator      | ""   ->  string | text          |

**transactions**

| key             | evmType      | pgType        |
| --------------- | ------------ | ------------- |
| blockNumber     | bigint       | bigint        |
| blockHash       | hex hash     | text          |
| _timestamp      | uint64       | bigint        |
| transactionHash | hex hash     | text          |
| _from           | address hash | text          |
| _to             | address hash | text          |
| gas             | uint64       | numeric(20,0) |
| gasUsed         | uint64       | numeric(20,0) |
| gasPrice        | big.Int      | numeric(20,0) |
| input           | []byte       | text          |
| logs            | log          | text          |
| nonce           | uint64       | numeric(20,0) |
| _value          | big.Int      | numeric(78,0) |
| contractAddress | address hash | text          |
| error           | string       | text          |
| decollator      | ""           | text          |

**traces**

| key           | evmType      | pgType        |
| ------------- | ------------ | ------------- |
| id            | bigint       | bigint        |
| _from         | address hash | text          |
| _to           | address hash | text          |
| _timestamp    | uint64       | bigint        |
| blockHash     | hex hash     | text          |
| blockNumber   | bigint       | bigint        |
| txHash        | hex hash     | text          |
| txIndex       | int          | integer       |
| pc            | int64        | bigint        |
| op            | string       | text          |
| gas           | uint64       | numeric(20,0) |
| gasCost       | uint64       | numeric(20,0) |
| depth         | int          | integer       |
| refundCounter | uint64       | numeric(20,0) |
| err           | string       | text          |
| decollator    | ""           | text          |

**precompiled**

| key           | evmType      | pgType        |
| ------------- | ------------ | ------------- |
| id            | bigint       | bigint        |
| _from         | address hash | text          |
| _to           | address hash | text          |
| _timestamp    | uint64       | bigint        |
| blockHash     | hex hash     | text          |
| blockNumber   | bigint       | bigint        |
| txHash        | hex hash     | text          |
| txIndex       | int          | bigint        |
| gas           | uint64       | numeric(20,0) |
| gasCost       | uint64       | numeric(20,0) |
| depth         | int          | integer       |
| refundCounter | uint64       | numeric(20,0) |
| err           | string       | text          |
| decollator    | ""           | text          |





**events**

| key              | evmType           | pgType  |
| ---------------- | ----------------- | ------- |
| id               | bigint            | bigint  |
| eventAddress     | address hash      | text    |
| topics           | []common.Hash(32) | text    |
| _timestamp       | uint64            | bigint  |
| eventData        | []byte            | text    |
| blockNumber      | bigint            | bigint  |
| transactionHash  | hex hash          | text    |
| transactionIndex | int               | integer |
| blockHash        | hex hash          | text    |
| logIndex         | int               | bigint  |
| removed          | bool              | boolean |
| decollator       | ""                | text    |

### 类型划分原则



1. 对于logIndex，txIndex，transferIndex，depth等都是使用int。

2. evm中为bigint类型的理论上应该设置成numeric(78,0)，`len(str(2**256))=78  len(str(2**64))=20`，但是对于一些特殊的bigint，如：blockNumber，id使用在EVM中使用bigint，在pg里面使用bigint就好。对于totalDiffculty，Difficult两个字段而言在EVM中为bigint，在pg中使用numeric(32,0)。

3. 对于EVM中为uint64类型的字段，一些字段在pg中的类型需要设置为numeric(20,0)，因为该值可能会超过bigint的int64的取值范围，例如：nonce，gas，gascost，gasused，gascost，gasprice，refundCounter。但是对于uint64的timestamp字段可以使用bigint类型。对于EVM中为int64类型的字段在pg中使用bigint。
4. hex hash 为34位的字符串（32+2）addresshash为22位的字符串（20+2），这些在pg中都设置为text类型