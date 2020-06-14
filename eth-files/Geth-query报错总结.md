---
title: "Geth Query报错总结"
date: 2020-01-09T17:23:56+08:00
draft: false
summary: "以太坊智能合约数据抽取和重放报错总结"
tags: [geth-query,blockchain,ethereum]
---

## Geth-query报错总结

> 本文主要介绍Geth-query工具编写过程中的一些问题，特此记录。

### 并发程序文件夹创建失败

对于对并发程序对于新建一个文件夹，正常逻辑是首先检查一个文件夹是否存在，不存在就创建一个文件夹。但是可能存在一种情况：在检查csv_data目录的时候可能会在检查`os.IsNotExist(err)`时候文件不存在，但是`os.Mkdir(absDir, 0777)`的时候该`csv_data`文件已经由别的协程创建成功，因此writer指针为空。

```golang
	if _, err := os.Stat(absDir); os.IsNotExist(err) {
		err = os.Mkdir(absDir, 0777)
		if err != nil {
			Logger
			return nil, err
		}
	}

	preDir := absDir + "/" + prefix
	if _, err := os.Stat(preDir); os.IsNotExist(err) {
		err = os.Mkdir(preDir, 0777)
		if err != nil {
			return nil, err
		}
	}

	folderDir := fmt.Sprintf("%v/%v", preDir, folderSeq*FILE_NUMBER_IN_FOLDER*BLOCK_NUMBER_IN_FILE)
	if _, err := os.Stat(folderDir); os.IsNotExist(err) {
		err = os.Mkdir(folderDir, 0777)
		if err != nil {
			return nil, err
		}
	}
```

### for range语句循环变量为值传递

对于情况1能够将`transfers`的地址加入切片，情况2将循环局部变量的地址加入切片，情况3新建一个对象将`transferTmp`的值赋值给新对象，并且将新对象的地址加入切片。

```golang
		for _, transferTmp := range transfers {
			transfer := (*valueTransfer)(unsafe.Pointer(&transferTmp))
			// 1.
			// data.transfers = append(data.transfers, (*valueTransfer)(unsafe.Pointer(&transfers[i])))
			// Logger.Infof("value: %v", data.transfers[len(data.transfers)-1].value)
			// 2.
			// data.transfers = append(data.transfers, (*valueTransfer)(unsafe.Pointer(&transfer)))
			data.transfers = append(data.transfers, &valueTransfer{transfer.depth, transfer.transactionHash, transfer.src, transfer.srcBalance, transfer.dest, transfer.destBalance, transfer.value, transfer.kind, transfer.snapshot})
		}
```

### go 指针内存释放问题

1. 对于全局变量要通过指针保存局部变量的值，最好的办法是将局部变量指针指向的局部地址内容重新new一遍，然后赋给全局变量指针。因为局部变量的值在后面可能发生变化。使用全局变量的指针保存局部变量的指针最后全局变量保存的值具有很大的随机性(**全局变量保存局部变量的某一成员指针，局部变量也会被gc释放**)。

```golang
		evm.Interpreter().(*EVMInterpreter).GetConfig().Tracer.(*StructLogger).CaptureTransferState(evm.StateDB.(*state.StateDB), evm.depth, common.Hash{}, caller.Address(),
			to.Address(), big.NewInt(0).Set(value), "CALL", evm.StateDB.(*state.StateDB).GetNextRevisionId())
```

上面一段代码中直接保存value的地址，该value地址的值后面可能会发生变化，尽管有指针副本一直指向该处，内存不会释放，但是其值已经改变。具体错误现象是`transfer`中的value，只有transactions中的value和gas fee的value是正确的。其余内部交易的value值都是错误的。因为stack中每次push的value值都是从`integer = interpreter.intPool.get()`中获取的，该pool中的value值是在动态改变。

2. 局部指针变量指向一块较大内存，通过全局指针保存。导致一直有指针指向局部分配的较大内存地址，因此大内存一直不能被go的gc释放，导致内存占用过高。

> 上述问题是由于captureState函数中新建了一些对象，然后这些对象的指针被保存在全局变量的结构体中，导致go gc工具在释放内存时由于captureState中的对象一直有全局对象指针指向其值，导致其内存一直得不到释放。stack和memory都是一个1024字节的数组，因此其内存一直就是成M的增加，当碰到6810086中的`0x68b71d202dc52ad80812b563f3f6b0aaf1f19c04c1260d13055daad5b88a36a8`交易时，其trace数量为100W，导致其内存分配巨大而一直得不到释放。只需将保存stack和memory的部分注释掉即可，因为该部分只要debug打开，该部分会默认加入到structlog的全局变量中。

### 其它问题reverTransfer 中的revertNum初始值应该为-1， 避免transfer全部revert掉之后初始值0，还剩余一个transfer。

2. selfdestruct中转账的源地址为合约地址，目的地址为stack.pop()出来的地址，注意stack.pop()动作不要为了获取目的地址而执行两遍。
3. 对于call更改上下文环境，使用上下文环境是调用合约的上下文环境(storage)，callcode和delegatecall不切换上下文环境，相当于把带有另外合约账户的代码。call和callcode改变msg.sender为当前的调用者，delegatecall的msg.sender一直都是最初的调用者。例子：用户A调用合约B，合约B调用合约C(callcode B和delegatecall A)

callcode和call 可能导致内部交易，staticcall和delegatecall不会导致内部交易。

4. selfdestruct 

   该交易的stack为空

   0xbb8ee9866ee67277986b6f40775469c7a674810ce99dce3caff0d1117c8dcdac

   这里校验失败:

   ```golang
   		if sLen := stack.len(); sLen < operation.minStack {
   			fmt.Printf("stack underflow (%d <=> %d)", sLen, operation.minStack)
   			return nil, fmt.Errorf("stack underflow (%d <=> %d)", sLen, operation.minStack)
   		} else if sLen > operation.maxStack {
   			fmt.Printf("stack limit reached %d (%d)", sLen, operation.maxStack)
   			return nil, fmt.Errorf("stack limit reached %d (%d)", sLen, operation.maxStack)
   		}
   ```

   解决方案：在logger.go中加入stack值大小的校验。

5. 对于预编译合约的调用使用call，该call会被traces表抓取，然后被precompileds表抓取。这两个过程是连续的，

6. 队以call，callcode，staticcall的from字段都是contract.Address()，但是delegatecall的from字段为contract.caller()字段，因为可能出现连续两个delegate的情况。通过`evm.go`的该行代码可以明确他们的区别`contract := NewContract(caller, to, value, gas)`。

   ```golang
   type Contract struct {
   	// CallerAddress is the result of the caller which initialised this
   	// contract. However when the "call method" is delegated this value
   	// needs to be initialised to that of the caller's caller.
   	调用该合约人的地址
   	CallerAddress common.Address
   	// 调用合约人的合约引用格式
   	caller        ContractRef
   	// 本合约的地址信息
   	self          ContractRef
   ```

   



7. 对于traces表不能完整反应外部账户和合约账户之间的调用情况。因为最外层的call和create交易不会被抓取到traces信息中，下面举两个例子：

首先观察创建合约转账交易`0xd1d0bdadc9b60a4d12184ea90142629fc1d2c26741451b75f2e226f0beb406f8`

在etherscan中没有create的trace信息。看该交易由geth-query工具抓取出来的traces信息：

```bash
     id      |                   _from                    | _to  | _timestamp |                             blockhash                              | blocknumber |                               txhash                               | txindex | pc  |   op   |  gas   | gascost | depth | refundcounter | err | decollator
-------------+--------------------------------------------+------+------------+--------------------------------------------------------------------+-------------+--------------------------------------------------------------------+---------+-----+--------+--------+---------+-------+---------------+-----+------------
 11652500011 | 0x6f5678935F3b6055A77A37b33F326d4534693918 | 0x00 | 1440088773 | 0xec46ee8db536b9bfe6c0721c9d98e231c9b4731acf60ae6e74f41645ad0354ac |      116525 | 0xd1d0bdadc9b60a4d12184ea90142629fc1d2c26741451b75f2e226f0beb406f8 |      70 |  78 | SSTORE | 234489 |   20000 |     1 |             0 |     |
 11652500012 | 0x6f5678935F3b6055A77A37b33F326d4534693918 | 0x00 | 1440088773 | 0xec46ee8db536b9bfe6c0721c9d98e231c9b4731acf60ae6e74f41645ad0354ac |      116525 | 0xd1d0bdadc9b60a4d12184ea90142629fc1d2c26741451b75f2e226f0beb406f8 |      70 | 100 | SSTORE | 214392 |   20000 |     1 |             0 |     |
 11652500013 | 0x6f5678935F3b6055A77A37b33F326d4534693918 | 0x00 | 1440088773 | 0xec46ee8db536b9bfe6c0721c9d98e231c9b4731acf60ae6e74f41645ad0354ac |      116525 | 0xd1d0bdadc9b60a4d12184ea90142629fc1d2c26741451b75f2e226f0beb406f8 |      70 | 143 | SSTORE | 194244 |   20000 |     1 |             0 |     |
(3 rows)
```

也没有create信息，上面的trace信息是创建合约时调用合约构造函数的trace。

观察内部交易，发现内部交易中有抓取出来的create transfer：

```bash
     id      | blocknumber |                             blockhash                              | _timestamp |                               txhash                               | transferindex | depth |                   _from                    |                    _to                     |    frombalance     |       tobalance        |   transfervalue   | transfertype | decollator
-------------+-------------+--------------------------------------------------------------------+------------+--------------------------------------------------------------------+---------------+-------+--------------------------------------------+--------------------------------------------+--------------------+------------------------+-------------------+--------------+------------
 11652500142 |      116525 | 0xec46ee8db536b9bfe6c0721c9d98e231c9b4731acf60ae6e74f41645ad0354ac | 1440088773 | 0xd1d0bdadc9b60a4d12184ea90142629fc1d2c26741451b75f2e226f0beb406f8 |           142 |     0 | 0xa9Ac1233699BDae25abeBae4f9Fb54DbB1b44700 | 0x6f5678935F3b6055A77A37b33F326d4534693918 | 977900000000000000 |                      0 |                 0 | CREATE       |
 11652500143 |      116525 | 0xec46ee8db536b9bfe6c0721c9d98e231c9b4731acf60ae6e74f41645ad0354ac | 1440088773 | 0xd1d0bdadc9b60a4d12184ea90142629fc1d2c26741451b75f2e226f0beb406f8 |           143 |     0 | 0xa9Ac1233699BDae25abeBae4f9Fb54DbB1b44700 | 0xe6A7a1d47ff21B6321162AEA7C6CB457D5476Bca | 966840800000000000 | 3161572539666965305125 | 11059200000000000 | GASFEE       |
(2 rows)
```



观察普通转账的交易`0x876653f5459ce949472d70ed9bdac596450ffa051fd0a58f7ec02cab152e47e1`该交易为一笔普通转账交易。在etherscan上没有trace信息，geth-qery工具抓取的traces信息如下：

```bash
 id | _from | _to | _timestamp | blockhash | blocknumber | txhash | txindex | pc | op | gas | gascost | depth | refundcounter | err | decollator
----+-------+-----+------------+-----------+-------------+--------+---------+----+----+-----+---------+-------+---------------+-----+------------
(0 rows)
```

然后查看该交易的transfer信息如下：

```bash
     id      | blocknumber |                             blockhash                              | _timestamp |                               txhash                               | transferindex | depth |                   _from                    |                    _to                     |     frombalance     |        tobalance        |    transfervalue    | transfertype | decollator
-------------+-------------+--------------------------------------------------------------------+------------+--------------------------------------------------------------------+---------------+-------+--------------------------------------------+--------------------------------------------+---------------------+-------------------------+---------------------+--------------+------------
 15000200000 |      150002 | 0x87444cdd8fa9502ef46b8edbdcefa7c199df2bd5efc939b363cb8aa9c26a9588 | 1440662073 | 0x876653f5459ce949472d70ed9bdac596450ffa051fd0a58f7ec02cab152e47e1 |             0 |     0 | 0x608f4b86321a68453f9adf6F06F7Fc3E621739ad | 0x32Be343B94f860124dC4fEe278FDCBD38C102D88 | 5000000008045985864 | 41313540674717582890816 | 4998781970000000000 | CALL         |
 15000200001 |      150002 | 0x87444cdd8fa9502ef46b8edbdcefa7c199df2bd5efc939b363cb8aa9c26a9588 | 1440662073 | 0x876653f5459ce949472d70ed9bdac596450ffa051fd0a58f7ec02cab152e47e1 |             1 |     0 | 0x608f4b86321a68453f9adf6F06F7Fc3E621739ad | 0xe6A7a1d47ff21B6321162AEA7C6CB457D5476Bca |         11706882864 |   869096440077134764317 |    1218026339103000 | GASFEE       |
(2 rows)
```

transfer表抓取到了普通转账交易的call内部交易信息。

**结论**

这里错误，单单依靠traces表不能反映外部账户和合约账户的联系，因为最外层交易的create和call信息并未记录在traces中。