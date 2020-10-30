# ckb-rollup

# 1. 背景

## 1.1. loopring

### 1.1.1. 背景

loopring是基于zkrollup的去中心化交易所，其通过零知识证明的方法解决了传统中心化交易所用户对自己资金不可控的风险，实现了“交易所作恶而不能获益”的目的，然后通过zkrollup的2层模型，在不降低安全性的前提下对交易所进行扩容，极大提升了交易所的tps以及降低了交易的手续费。

### 1.1.2. loopring-circuit3.5版本

当前`loopring-circuit`的版本为3.5版本，相比于3.0版本主要做了以下的改进：

#### 业务逻辑方面

1. 去除了`orderCancellationCircuit`，新建了链下内部交易电路。
2. 抽象出了`circuit` 抽象类，通过实现该抽象类，进而实现了链上存款，链上退款，链上环匹配，以及链下内部转账，链下退款5条电路。
3. 区分了`Mode::CreateKeys`和`Mode::Prove`两种模式，之前的代码中但执行`Mode::Prove`模式时，如果当前是`Mode::Prove`模式但是没有生成pk和vk，首先应该进行pk和vk的生成。当前版本当生成证明的时候不生成pk和vk。
4. 在`main.cpp`中将服务器的启动模式通过枚举类型进行划分，通过环境变量传入不同的启动模式。

#### 性能优化方面

1. 添加GPU证明优化。
2. 证明电路代码优化。主要有vector的提前reserve；抽象出`circuit`抽象类，并且在抽象类中添加打印调试信息；如果是多核cpu，`#pragma omp parallel for`并行优化`generate_r1cs_witness` 的for循环。
3. 底层算法库的优化。(暂时未深入) 

### 1.1.3. merkleTree状态树设计

```c++
// 当前merkletree为4叉树
// 账户节点，高度为12，一共能够容纳4^(12-1)=4194304个账户节点
class Account
{
public:
    ethsnarks::jubjub::EdwardsPoint publicKey;
    ethsnarks::FieldT nonce;
    ethsnarks::FieldT balancesRoot;
};

// balance的叶子节点，高度为5，一共能够容纳4^(5-1)=256个tokenId
class BalanceLeaf
{
public:
    ethsnarks::FieldT balance;
    ethsnarks::FieldT tradingHistoryRoot;
};

// balance下面的挂单，高度为7，一共能够容纳4^(7-1)=4096个匹配订单
class TradeHistoryLeaf
{
public:
    ethsnarks::FieldT filled;
    ethsnarks::FieldT orderID;
};

// 订单
class Order
{
public:
    ethsnarks::FieldT exchangeID;
    ethsnarks::FieldT orderID;
    ethsnarks::FieldT accountID;
    ethsnarks::FieldT tokenS;
    ethsnarks::FieldT tokenB;
    ethsnarks::FieldT amountS;
    ethsnarks::FieldT amountB;
    ethsnarks::FieldT allOrNone;
    ethsnarks::FieldT validSince;
    ethsnarks::FieldT validUntil;
    ethsnarks::FieldT maxFeeBips;
    ethsnarks::FieldT buy;

    ethsnarks::FieldT feeBips;
    ethsnarks::FieldT rebateBips;

    Signature signature;
};

```

### 1.1.4. 业务电路分析

> 链上存款，链上退款，链下退款，链下内部转账为四条基础的业务逻辑电路，链上环匹配为特定业务逻辑的电路，因此在本文中主要对loopring的前面四种电路的实现进行分析。

#### 1.1.4.1. 链上存款

##### 整体业务流程

1. 用户发送Deposit请求到ETH主链的XXX合约。
2. 中继监听XXX合约，收到用户的Deposit请求。
3. 中继组装Deposit请求生成Deposit请求的block进行提交，调用XXX合约的commitBlock。
4. 中继调用电路部分的block的prove。
5. 中继调用XXX合约的VerifiyBlock。

![deposit流程](C:\Users\czm18\Desktop\deposit流程.png)



```
上图中各个步骤的数据结构如下：
1. function deposit(
        ExchangeData.State storage S,
        address from,
        address to,
        address tokenAddress,
        uint96  amount,                 // can be zero
        bytes   memory extraData
        )
        internal  // inline call
        
2. DepositRequested(
	from,
	to,
	tokenAddress,
	tokenID,
	amountDeposited)

3. depost_block 该部分的格式由于中继未公开所以展示未知。

4. block.json 详细格式参见protocols/packages/loopring_v3/circuit/test/data/block.json

5. proof (a, b, publicData)

6. submitBlocks() 
// This is the (virtual) block the owner  needs to submit onchain to maintain the
    // per-exchange (virtual) blockchain.
    struct Block
    {
        uint8      blockType;
        uint16     blockSize;
        uint8      blockVersion;
        bytes      data; // publicData，链上会通过publicData计算其hash，然后proof+publicInput+vk进行区块正确性的验证。
        uint256[8] proof; // circuit生成的proof

        // Whether we should store the @BlockInfo for this block on-chain.
        bool storeBlockInfoOnchain; 

        // Block specific data that is only used to help process the block on-chain.
        // It is not used as input for the circuits and it is not necessary for data-availability.
        AuxiliaryData[] auxiliaryData; // 每笔交易的额外信息，但是这些信息在数据可用性中不需要，这些数据仅仅帮助处理区块上链

        // Arbitrary data, mainly for off-chain data-availability, i.e.,
        // the multihash of the IPFS file that contains the block data.
        bytes offchainData; // 链下的数据可用性，虚拟块在IPFS系统中的散列
    }
    
    // General auxiliary data for each conditional transaction
    struct AuxiliaryData
    {
        uint  txIndex;
        bytes data;
    }
```

##### 电路设计图

![1601454483093](C:\Users\czm18\AppData\Roaming\Typora\typora-user-images\1601454483093.png)

```C++
// DepositBlock
class DepositBlock
{
public:
    ethsnarks::FieldT exchangeID;
    // 状态树起始的根节点
    ethsnarks::FieldT merkleRootBefore;
    ethsnarks::FieldT merkleRootAfter;
    
    // onchain请求累计的起始hash
    // # Accumulated hash of starting onchain request
    libff::bigint<libff::alt_bn128_r_limbs> startHash;
    
    // # Index of onchain request
    // onchain请求开始的索引
    ethsnarks::FieldT startIndex;
    // # Number of onchain requests processed in this block
    // 在该区块中的Deposit请求数量
    ethsnarks::FieldT count;

    std::vector<Loopring::Deposit> deposits;
};

// Deposit
class Deposit
{
public:
    // 存款数目
    ethsnarks::FieldT amount;
    // balanceRoot更新
    BalanceUpdate balanceUpdate;
    // accountRoot更新
    AccountUpdate accountUpdate;
};

```



##### 电路运行步骤概述

1. 计算存储金额的上限，如果存储金额后总金额大于最大金额容量2**96-1，则存储后金额为最大金额。

2. updateBalance

3. updateAccount

   

#### 1.1.4.2. 链上退款

##### 整体业务流程

1. 用户发送withdraw请求到ETH主链的Exchange合约
2. 中继监听Exchange合约收到用户的withdraw请求
3. 中继组装withdraw请求生成withdraw的block提交，调用Exchange的commitBlock
4. 中继调用电路部分生成block的prove
5. 中继调用Exchange的verifyBlock

![onchainWithdraw流程](C:\Users\czm18\Desktop\onchainWithdraw流程.png)

```
1. withdraw 部分分为很多种withdraw
forceWithdraw 强制性的退款，一次性能够退款该账户的所有费用，但是要交手续费
        address owner,
        address token,
        uint32 accountID
        
withdrawFromMerkleTree 在退款模式种通过验证MerkleTree的方式来提出余额
ExchangeData.MerkleProof calldata merkleProof

withdrawFromDepositRequest 从pendingDeposit种进行提款
        address owner,
        address token
        
withdrawFromApprovedWithdrawals 从批准的金额中进行提款
        address[] memory owners,
        address[] memory tokens
        
distributeWithdrawal 尝试退款指定金额，如果退款失败就将退款金额加入到批准的退款金额中
        address from,
        address to,
        uint16 tokenID,
        uint256 amount,
        bytes memory extraData,
        uint256 gasLimit
        
2. emit 一共分为分为3种
除了forcewithdraw emit以下数据之外
event ForcedWithdrawalRequested(
        address owner,
        address token,
        uint32 accountID
    );
别的withdraw数据都是withdraw以下数据
event WithdrawalCompleted(
        address from,
        address to,
        address token,
        uint    amount
    );

    event WithdrawalFailed(
        address from,
        address to,
        address token,
        uint    amount
    );

3. withdraw_block 暂时未知。

4. block.json 见上面文件夹

5. proof

6. submitBlocks和上面的deposit的数据一样。
```



##### 电路设计图

![1601454564695](C:\Users\czm18\AppData\Roaming\Typora\typora-user-images\1601454564695.png)

```C++
class OnchainWithdrawalBlock
{
public:
    ethsnarks::FieldT exchangeID;

    ethsnarks::FieldT merkleRootBefore;
    ethsnarks::FieldT merkleRootAfter;

    ethsnarks::LimbT startHash;

    ethsnarks::FieldT startIndex;
    ethsnarks::FieldT count;

    std::vector<Loopring::OnchainWithdrawal> withdrawals;
}

class OnchainWithdrawal
{
public:
    // 退款金额
    ethsnarks::FieldT amountRequested;
    // 更新balanceRoot
    BalanceUpdate balanceUpdate;
    // 更新accountRoot
    AccountUpdate accountUpdate;
};
```



##### 电路运行步骤概述

因为是链上退款操作，需要判断当前是否是bshutdownmode，如果是shutdownmode，用户退款会直接将该用户账户的所有余额直接退款，并且将状态树上对用的节点回执为空。并且对于链上退款操作，operator是不收取手续费用。

**onchainWithdrawGadget**

1. 根据账户的余额以及当前的退款值，计算当前实际的退款值。
2. 在bshutdownmode模式下直接退款当前账户余额，在非该模式下，将当前的退款值转化为float类型，进行退款操作。
3. 在bshutdownmode模式下将状态树的一些状态执为初始值。
4. 更新账户的balanceRoot以及accountRoot。

**onchainWithdrawCircuit**

1. 根据每一笔withdraw操作修改账户的balanceRoot以及AccountRoot，并且计算数据的累计哈希（链上存款和链上撤款都需要计算累计哈希，并且将最后的累计哈希结果添加到publicData中）。
2. 计算publicData。
3. 校验计算的状态树root和期待状态树的root是否相同。



#### 1.1.4.3. 链下退款

##### 整体业务流程

1. ⽤户发送withdraw请求到中继
2. 中继收到⽤户withdraw请求。组装withdraw请求⽣成withdraw的block提交到ETH主链，调⽤Exchange的commitBlock
3. 中继调⽤电路部分⽣成block的prove
4. 中继调⽤Exchange合约的verifyBlock

前面两个电路都是onchain的一些操作，下面讨论的电路都是一些offchain的操作，即用户直接向operator发送对应的请求，然后operator收到对应的请求之后，打包提交证明。

![offchainwithdraw](C:\Users\czm18\Desktop\offchainwithdraw.png)

```
1. withdraw 格式未知

2. withdraw_block 暂时未知。

3. block.json 见上面文件夹

4. proof

5. submitBlocks和上面的deposit的数据一样。
```



##### 电路设计图

![1601454660297](C:\Users\czm18\AppData\Roaming\Typora\typora-user-images\1601454660297.png)

```C++
class OffchainWithdrawalBlock
{
public:
    // 交易所ID
    ethsnarks::FieldT exchangeID;

    ethsnarks::FieldT merkleRootBefore;
    ethsnarks::FieldT merkleRootAfter;
    
    // 所有数据累计hash
    ethsnarks::LimbT startHash;
    
    // operator账户ID
    ethsnarks::FieldT operatorAccountID;
    // 更新operator账户
    AccountUpdate accountUpdate_O;

    std::vector<Loopring::OffchainWithdrawal> withdrawals;
};

class OffchainWithdrawal
{
public:
    // 退款金额
    ethsnarks::FieldT amountRequested;
    // 费用
    ethsnarks::FieldT fee;
    Signature signature;
    
    // 账户A费用token，退款金额余额更新，账户更新
    BalanceUpdate balanceUpdateF_A;
    BalanceUpdate balanceUpdateW_A;
    AccountUpdate accountUpdate_A;
    // operator费用更新
    BalanceUpdate balanceUpdateF_O;
};

```

##### 电路运行步骤概述

每个withdrawal的逻辑跟onchainwithdraw的逻辑类似，但是不需要判断shutdownmode，增加了fee的操作以及验证用户的签名等逻辑，并且需要付给operator支付费用。

**offchainWithdrawGadget**

1. 将交易费用转换为float类型，并且做精确度校验。
2. 用户给operator支付退款费用。
3. 根据当前用户balance以及退款金额得出实际的退款金额。
4. 将退款值转换为float类型，并且指定精确度。
5. 计算退款账户新的账户余额。
6. 用户的nonce值加1。
7. 更新用户费用账户的balanceRoot。
8. 更新用户金额账户的balanceRoot。
9. 更新account的根节点。
10. 更新operator费用账户的balanceRoot。

**offchainWithdrawCircuit**

1. 检查operator账户不为空。
2. 根据每笔withdraw更新accountRoot，以及更新operator费用账户的balanceRoot。
3. 最后右operator的费用balanceRoot更新accountRoot。
4. 根据dataAvaiable模式计算publicData。
5. 检查电路计算的merkleRoot和期待的merkleRoot是否相等。

#### 1.1.4.4. 链下转账

##### 整体业务流程

1. ⽤户发送InternalTransfer请求到中继
2. 中继收到⽤户InternalTransfer请求。组装InternalTransfer请求⽣成InternalTransfer的block提交到ETH主链，调⽤Exchange的commitBlock
3. 中继调⽤电路部分⽣成block的prove
4. 中继调⽤Exchange合约的verifyBlock

![offchaintransfer](C:\Users\czm18\Desktop\offchaintransfer.png)

```
1. withdraw 格式未知

2. withdraw_block 暂时未知

3. block.json 见上面文件夹

4. proof

5. submitBlocks和上面的deposit的数据一样。
```



##### 电路设计图

![InternalTransferGadgetCircuit](C:\Users\czm18\Downloads\InternalTransferGadgetCircuit.png)

```C++
class InternalTransfer
{
public:
    ethsnarks::FieldT fee;
    ethsnarks::FieldT amount;
    ethsnarks::FieldT type;
    Signature signature;

    ethsnarks::FieldT numConditionalTransfersAfter;
    // from账户状态更新
    BalanceUpdate balanceUpdateF_From; // pay fee step
    BalanceUpdate balanceUpdateT_From; // transfer step
    AccountUpdate accountUpdate_From;
    
    // to账户状态更新
    BalanceUpdate balanceUpdateT_To;   // receive transfer
    AccountUpdate accountUpdate_To;
    // operator费用更新
    BalanceUpdate balanceUpdateF_O;	   // receive fee
};


class InternalTransferBlock
{
public:
    // 交易所ID
    ethsnarks::FieldT exchangeID;

    ethsnarks::FieldT merkleRootBefore;
    ethsnarks::FieldT merkleRootAfter;
    // operator账户ID
    ethsnarks::FieldT operatorAccountID;
    // 更新operator的accountRoot
    AccountUpdate accountUpdate_O;

    std::vector<Loopring::InternalTransfer> transfers;
};
```



##### 电路运行步骤概述

**InternalTransferGadget**

1. 对from账户发起的交易进行验签操作。
2. 确保to账户地址一定存在。
3. 确定交易费用，并且将交易费用转化为float类型。
4. 确定交易金额，并且将交易金额转化为float类型。
5. from地址支付交易费用。
6. from地址给to地址支付转账金额。
7. from地址的nonce加1。
8. 更新from和to的费用balanceRoot，余额的balanceRoot，最后更新其accountRoot。
9. 更新operator的balanceRoot。

**InternalTransferCircuit**

1. 对operator账户是否存在进行校验。
2. 根据每笔transfer计算accountRoot的变化，以及operator账户balanceRoot的变化。
3. 根据最后operator账户的balanceRoot计算accountRoot。
4. 计算publicData

#### 1.1.4.5. 环结算

##### 整体业务流程

1. 用户发送match order请求到中继。
2. 中继收到用户的match order请求，在线下匹配订单，并将订单两两组成`RingSettlement`，然后组成`RingSettlementBlock`提交到ETH主链，调用XXX合约commitblock。
3. 中继调用电路部分生成block的prove。
4. 中继调用XXX合约验证verifyBlock。

##### 电路设计图

![1601455209179](C:\Users\czm18\AppData\Roaming\Typora\typora-user-images\1601455209179.png)

```C++
class RingSettlementBlock
{
public:
    // 交易所的ID
    ethsnarks::FieldT exchangeID;

    ethsnarks::FieldT merkleRootBefore;
    ethsnarks::FieldT merkleRootAfter;
    // timestamp used in this block
    ethsnarks::FieldT timestamp;
    
    // Protocol fees used in this block
    ethsnarks::FieldT protocolTakerFeeBips;
    ethsnarks::FieldT protocolMakerFeeBips;
    
    // operator sign
    // operator 打包区块签名
    Signature signature;
    // protocol fee account update data
    // 更新接收协议费用的账户状态
    AccountUpdate accountUpdate_P;
    
    ethsnarks::FieldT operatorAccountID;
    // operator account update
    // 更新operator账户状态
    AccountUpdate accountUpdate_O;

    std::vector<Loopring::RingSettlement> ringSettlements;
};

class RingSettlement
{
public:
    Ring ring;
    
    // the starting merkleroot
    ethsnarks::FieldT accountsMerkleRoot;
    
    // update tradeHistroy
    // 更新账户A和B的tradingHistory
    TradeHistoryUpdate tradeHistoryUpdate_A;
    TradeHistoryUpdate tradeHistoryUpdate_B;
    
    // update accountA information
    // 更新账户A的balance以及account信息
    BalanceUpdate balanceUpdateS_A;
    BalanceUpdate balanceUpdateB_A;
    AccountUpdate accountUpdate_A;
    
    // update accountB information
    // 更新账户B的balance以及account信息
    BalanceUpdate balanceUpdateS_B;
    BalanceUpdate balanceUpdateB_B;
    AccountUpdate accountUpdate_B;
    
    // balance update data for protocol fee payment
    // 更新协议费用给协议费用账户
    BalanceUpdate balanceUpdateA_P;
    BalanceUpdate balanceUpdateB_P;
    
    // balance update data for the operator(fee/rebate/protocol fee)
    // 更新操作员收到的费用
    BalanceUpdate balanceUpdateA_O;
    BalanceUpdate balanceUpdateB_O;
};

```

##### 电路运行步骤概述

1. 使用`OrderMatchingGadget`匹配两个订单，获得订单匹配结果，如果订单能够匹配上，获取本次匹配的填充量。
2. 校验填充值`fillS_A`使用`floatGadget`编码的精度损失，精度损失需要在`Float24Accuracy`范围内。
3. 根据`buy`标识识别谁是`maker`和谁是`taker`，然后计算实际`taker`的买入值和`maker`的卖出值。
4. 计算A和B订单填充之后的值。
5. 计算订单A和B的协议费用。
6. 订单A和B都减少卖出的`fillS_value`。
7. 用户给`operator`支付总费用。
8. `operator`分别给订单AB回扣和支付`protocol fee`。
9. 更新A,B,protocol account, operator的merkletree状态。

### 1.1.5. loopring-circuit3.6版本

loopring3.6电路版本取消了blockType的概念，一个通用区块中可以包含多笔不同类型的交易，将所有电路抽象为`UniversialCircuit`，将所有的交易抽象为`transactionGadget`，这样做的好处是：在tps较低的情况下可能获得较好的性能，避免因区块不满而等待交易增加延迟，或者过快提交交易数量较少的区块造成较高手续费以及增加以太坊的拥塞情况。综合交易所当前的运行情况，因此选用该种方案。

电路的依赖关系：

```c++
       Circuit         --->      UniversalCircuit依赖于TransactionGadget，TransactionGadget通过selectTransactionGadget的输入来初始化电路。

                                 selectTransactionGadget
                                 AccountUpdateCircuit
                                 AmmUpdateCircuit
BaseTransactionCircuit --->      DepositCircuit
                                 NoopCircuit
                                 spotTradeCircuit
                                 TransferCircuit
                                 UniversalCircuit
                                 WithdrawCircuit      
```

#### 电路列表

| 电路名称       | 功能                       |
| -------------- | -------------------------- |
| Spot trade     | 现货交易（ring settlement) |
| Transfer       | 内部转账                   |
| Deposit        | 存款                       |
| Withdraw       | 撤款                       |
| update account | 更新账户                   |
| update AMM     | 打开账户自动做市函数       |
| No-op          | 将数据可用性的设置为0      |
|                |                            |

订单取消，强制退款，创建账户等操作可以从上面的操作中进行派生。

#### 抽象电路(UniversalCircuit)

`universalCircuit`是所有电路的抽象，该抽象电路中包含许多抽象的交易，抽象电路只对input数据，协议费用以及操作员账户的状态更新进行约束，其并不涉及每笔交易的约束构建，该部分约束构建交给抽象交易的组件进行完成。

##### 电路设计

![universalCircuit](C:\Users\czm18\Desktop\universalCircuit.png)

##### 电路运行步骤

1. Input数据约束构建。
2. operator nonce_after自增。
3. 构建每笔交易的约束。
4. 更新协议费用账户的状态。
5. 更新操作员账户的状态。
6. 验证operator对区块的签名。
7. 校验状态merkleRoot是否等于期望的root。

####  抽象交易(TransactionGadget)

`TransactionGadget`是所有类型交易公共部分的抽象，抽象交易主要是抽象各类交易的公共部分的约束，将公共部分的约束放到同一个gadget中，然后将对应的withness输入部分暴露给不同的电路，不同的电路只负责公共约束的withness输入，对于传入的input而言，不同类型的交易会有不同类型的input。

##### 电路设计

![transactionGadget](C:\Users\czm18\Desktop\transactionGadget.png)

##### 电路运行步骤

1. 初始化交易状态(`transactionState`)。
2. 通过交易状态参数初始化各种类型电路值。
3. 通过selector实例化当前抽象交易为某一具体交易，并且根据交易类型选择构建约束。
4. 验证账户A和B的地址都存在(值为非0)。
5. 检查账户A和账户B的签名。
6. 更新账户A的storageRoot。
7. 更新账户A余额S的balanceRoot。
8. 更新账户A余额B的balanceRoot。
9. 更新账户A的accountRoot。
10. 更新账户B的storageRoot。
11. 更新账户B余额S的balanceRoot。
12. 更新账户B余额B的balanceRoot。
13. 更新账户B的accountRoot。
14. 更新operator余额B的balanceRoot。
15. 更新operator余额A的balanceRoot。
16. 更新operator账户的accountRoot。
17. 更新协议费用账户余额B的balanceRoot。
18. 更新协议费用账户余额A的balanceRoot。



#### 基本交易电路(BaseTransactionCircuit)

通过transactionState对象，将所有varivale都保存到uOutputs以及aOutputs的vector中，也就是将抽象交易的所有输入引脚(输入变量)都保存在该类的成员变量中。

#### 内部转账(transfer)

内部转账电路支持链下2层转账过程，用户向链下中继发送内部转账请求，中继打包请求并且提交区块到合约，然后中继对区块在链下生成证明，并且提交生成的证明到链上进行验证。

##### 电路设计



##### 电路运行步骤

1. 检查input输入值是否有效。
2. 如果key没有给出，填写标准的双重作者key。
3. 验证支付者以及双重签名者的签名。
4. 构建账户A，B以及operator的账户约束。
5. 检查to账户的owner权限。
6. 通过约束条件进行一些优化。
7. 获取费用的精确度。
8. 获取amount的精确度。
9. 支付费用从from到operator。
10. 进行A和B之间的转账操作。
11. 统计type为1的交易数量。



### 1.1.6 合约部分设计

#### 主要的合约列表

| 合约              | 功能                                  |
| ----------------- | ------------------------------------- |
| universalRegister | 协议注册和管理入口                    |
| LoopringV3        | loopring协议合约                      |
| ExchangeV3        | Exchange合约                          |
| BlockVerifier     | 各种类型block验证器                   |
| ExchangeProxy     | 利用Proxy机制为Exchange升级提供可能性 |
| UserStakingPool   | 处理用户stake的相关逻辑               |
| ProtocolFeeVault  | 协议费用保管合约                      |
| LzDecompressor    | 数据压缩合约                          |
|                   |                                       |

#### 合约运行流程

ExchangeDeposits合约：用户进行存款操作 

ExchangeWithdraw合约：用户进行退款操作

ExchangeV3合约： 主要是operator的接口





### 1.1.7. 小结

loopringV3.5的电路代码设计完备，我们在设计时可以借鉴其链上存款，链上退款，链下退款，链下内部交易四条电路的方法，然后根据这四条电路的设计逻辑完善当前zkp-ckb库中缺少的gadget，最终完成以上电路的设计。



## 1.2. zksync

//TODO

## 1.3. zk-rollup-ckb



#### 1.3.1 总架构图

![total](C:\Users\czm18\Desktop\total.png)

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
    from // from的地址
    fromAccountID // from在merkletree的accountID
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
    To
    ToAccountID
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
	own
	ownAccountId
	publicKey
	nonce
}
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
}
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
    map<blockNumber, blockData> // 所有blockData的值
    blockNumberHeight //当前区块的高度
    pendingDeposit<address, Deposit> // 当前等待存款的数目
    pendingWithdraw<address, withdraw> // 当前等待撤款的数目

Deposit:
    from: uint32
    fromAccountId: uint8
    amount: uint96
    timestamp: uint64

withdraw:
    from: uint32
    fromAccountId : uint8
    amount: uint96
    timestamp: uint64

blockData:
    blockType: uint8
    blockNumber: uint64 // blockNumber 已经为key值
    blocksize: uint16
    blockVersion: uint8
    data: bytes
    proof: uint256[8]

___________________________________________________________
func:
deposit()
withdraw()
submitBlock() -> process() -> 分别处理deposit withdraw 
```

#### 2.3.7 线下状态计算程序（operator）

线下状态计算程序：

1. 监听链上的事件信息。
2. 监听用户发出的请求。
3. 使用交易池里面的交易打包块并且提交到链下程序计算提交到电路中的block.json。
4. 使用block.json文件生成proof。
5. 提交proof到链上进行验证。

该程序主要用户接收链上合约发出的onchain操作以及用户发出的offchain操作，通过将所有操作的交易汇集到交易池中，然后从交易池中取出各种交易打包成各种类型的块，然后将各种类型的区块丢到线下状态计算的程序中，计算出区块运算的中间状态，并且将中间状态作为电路的withness输入，输入到电路中进行证明的生成。

```
tx_pool
listen_contract_event():
    match txType {
        deposit:
        	processDeposit();
        	tx_pool.addTx();
        withdraw:
        	processWithdraw();
        	tx_pool.addTx();
    }
    
package_block_from_tx_pool():
	if (tx.getDepositSize() > block.size()) {
		package_deposit_block();
	} else if (tx.getwithdrawSize() > block.size()) {
		package_withdraw_block();
	} else if  (tx.getupdateAccountSize() > block.size()) {
		package_updateAccount_block();
	} else if (tx.getTransferSize() > block.size()) {
		package_transfer_block();
	}
    write_block_json()
    
  
 register_vk() {
 	
 }
 
 submit_block(){
 	proof, data
 }
 
 XXXblockData:
     operatorID: uint16
     merkleRootBefore: String
     merkleRootAfter: String
     blockNumber: uint64
     blockType: String
     timestamp: uint64
     vector<XXXtransaction>  // XXXtransaction中的内容就是用户发出的交易数据，并且这些交易数据都会压缩后上链
 
根据pool里面的各类交易数量，打包生成链下状态计算的block，然后根据该block生成带有中间数据的block.json文件 
 
    
```



#### 2.3.8 circuit

电路部分作为一个独立的程序，通过读取block.json的配置文件，然后根据不同的blockType调用不同的电路生成证明，最后将生成的证明结果输出到proof文件中。

```
setup();
depositCircuit(cs, depositBlockWithness);
withdrawCircuit(cs, withdrawBlockWithness);
updateAccountCircuit(cs, updateAccountBlockWithness);
transferAccountCircuit(cs, transferBlockWithness);
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

depositBlockWithness:
     operatorID: Field
     merkleRootBefore: Field
     merkleRootAfter: Filed
     blockNumber: Filed
     blockSize: Filed
     deposits: vector<depositWithness> 

depositWithness:
	accountMerkleRootBefore // 之前的merkleRoot
	accountUpdateA // 更新merkleRoot
	
	// publicData
	from // from的地址
	fromAccountID // from在merkletree的accountID
	amount
 
withdrawBlockWithness:
     operatorID: Field
     merkleRootBefore: Field
     merkleRootAfter: Filed
     blockNumber: Filed
     blockSize: Filed
     withdraws: vector<withdrawWithness> 

withdraWithness:
	accountMerkleRootBefor
	accountupdateA
	// publicData
	from
	fromAccountId
	amount
	

updateAccountBlockWithness:
     operatorID: Field
     merkleRootBefore: Field
     merkleRootAfter: Filed
     blockNumber: Filed
     blockSize: Filed
     updateAccounts: vector<updateAccountWithness> 

updateAccountWithness:
	accountMerkleRootBefor
	accountupdateA
	
	// publicData
	from
	fromAccountId
	publicKeyX
	publicKeyY


transferBlockWithness:
     operatorID: Field
     merkleRootBefore: Field
     merkleRootAfter: Filed
     blockNumber: Filed
     blockSize: Filed
     transfers: vector<transferWithness> 

transferWithness:
	accountMerkleRootBefor
	accountupdateA
	accountupdateB
	
	// publicData
	from
	fromAccountId
	to
	toAccountId
	amount
```



```json
{
    "operatorID": "123456",
    "blockType": 0,
    "blockNumber": 123,
    "timestamp": 1234567,
    "merkleRootBefore": "",
    "merkleRootAfter": "",
    "transactions": [
        {
            "withness": {
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
            "deposit": {
                "from": "",
                "fromAccountID": "",
                "amount": "",
                "txType": ""
            }
        },
        {
            "withness": {
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
            "accountupdate": {
                "from": "",
                "fromAccountId": "",
                "publicKeyX": "",
                "publicKeyY": "",
                "txType": ""
            }
        },
        {
            "withness": {
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
            "withness": {
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











