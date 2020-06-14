1. 对于对并发程序对于新建一个文件夹，正常逻辑是首先检查一个文件夹是否存在，不存在就创建一个文件夹。但是可能存在一种情况

```golang

[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:19 <ether-query>csv dir = /mnt/intel-disk/czm/csv_data_test
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:19 <ether-query>debug level = info
service &{0xc000119b80 0xc0004523c0 0xc00000c1e0 0xc00cf38000 0xc00cff60e0 <nil> 0xc00000e920 0xc0004856e0 0xc00044a000 0xc00035f930 0xc000452420 0xc000112500 0xc00cffa200 0xc0001c34a0 0xc0002aa880 [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] 1 <nil> {{0 0} 0 0 0 0}}, err <nil>2019/12/27 15:58:26 Starting etherquery service.
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>debug level = info
=========start consumBlock...
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>chain.CurrentBlock() 0x386400d965a0fa3cc08c6abfeaae02792391c6e7a52ebb5606832095dd8edc66
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>chain.CurrentBlock().Number().Uint64() 8789919
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>start block = 1735000, end block = 1750000
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>chain.CurrentBlock() 0x386400d965a0fa3cc08c6abfeaae02792391c6e7a52ebb5606832095dd8edc66
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>chain.CurrentBlock() 0x386400d965a0fa3cc08c6abfeaae02792391c6e7a52ebb5606832095dd8edc66
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>chain.CurrentBlock() 0x386400d965a0fa3cc08c6abfeaae02792391c6e7a52ebb5606832095dd8edc66
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>chain.CurrentBlock().Number().Uint64() 8789919
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>chain.CurrentBlock() 0x386400d965a0fa3cc08c6abfeaae02792391c6e7a52ebb5606832095dd8edc66
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>chain.CurrentBlock().Number().Uint64() 8789919
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>chain.CurrentBlock() 0x386400d965a0fa3cc08c6abfeaae02792391c6e7a52ebb5606832095dd8edc66
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>start block = 1734000, end block = 1734999
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>chain.CurrentBlock().Number().Uint64() 8789919
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>exporter table name = blocks
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>chain.CurrentBlock() 0x386400d965a0fa3cc08c6abfeaae02792391c6e7a52ebb5606832095dd8edc66
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>exporter table name = blocks
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>chain.CurrentBlock().Number().Uint64() 8789919
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>start block = 1729000, end block = 1729999
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>exporter table name = transactions
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>exporter table name = transactions
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>start block = 1728000, end block = 1728999
[^[[1;32mINFO ^[[0m] 2019-12-27 15:58:26 <ether-query>start block = 1722000, end block = 1722999
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x18 pc=0xc382c7]

goroutine 265 [running]:
gethQuery/etherquery.(*CSVWriter).write(0x0, 0xc00d15e000, 0xe, 0xe, 0xc00d0db900, 0x41f99b)
        /home/czm/gethQuery/etherquery/csvWriter.go:125 +0x37
gethQuery/etherquery.(*transferExporter).writePrefix(0xc00d0e4020, 0x0, 0xf5fa6b)
        /home/czm/gethQuery/etherquery/transfer.go:85 +0x82
gethQuery/etherquery.(*EtherQuery).consumeBlocks(0xc00cffa220, 0x1a7570, 0x1a7957)
        /home/czm/gethQuery/etherquery/etherquery.go:141 +0xaad
created by gethQuery/etherquery.(*EtherQuery).Start
        /home/czm/gethQuery/etherquery/etherquery.go:293 +0x2ec
```

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



在检查csv_data目录的时候可能会在检查`os.IsNotExist(err)`时候文件不存在，但是`os.Mkdir(absDir, 0777)`的时候该`csv_data`文件已经由别的协程创建成功，因此writer指针为空。



2. for range语句局部变量是值传递，不是地址传递。在代码中对于unsafe.Pointer尽量少用。



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

上面1没有问题，主要是被指针坑了。

```golang
		evm.Interpreter().(*EVMInterpreter).GetConfig().Tracer.(*StructLogger).CaptureTransferState(evm.StateDB.(*state.StateDB), evm.depth, common.Hash{}, caller.Address(),
			to.Address(), big.NewInt(0).Set(value), "CALL", evm.StateDB.(*state.StateDB).GetNextRevisionId())
```

报错：

```golang 
[INFO ] 2020-01-02 17:31:2 <ether-query>&{0 [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] [136 27 10 78 156 85 208 142 49 216 211 192 34 20 77 117 164 84 33 28] 0xc0010ccd00 [144 44 162 123 8 213 132 66 182 123 180 250 152 141 227 81 70 138 181 159] 0xc0010cd380 0xc0010cd3c0 GASFEE 0}
[INFO ] 2020-01-02 17:31:2 <ether-query>depth: 1, from 0x109C4f2CCc82C4d77Bde15F306707320294Aea3F, to 0x834e9B529AC9Fa63B39A06f8D8C9B0D6791FA5dF, transfer value 762, kind CALL
[INFO ] 2020-01-02 17:31:2 <ether-query>&{1 [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] [16 156 79 44 204 130 196 215 123 222 21 243 6 112 115 32 41 74 234 63] 0xc0010cca20 [131 78 155 82 154 201 250 99 179 154 6 248 216 201 176 214 121 31 165 223] 0xc0010cca60 0xc000f6c840 CALL 7}
[INFO ] 2020-01-02 17:31:2 <ether-query>depth: 1, from 0x109C4f2CCc82C4d77Bde15F306707320294Aea3F, to 0xFD2605a2bF58fDbB90db1Da55dF61628B47F9e8c, transfer value 749630779616631718155781435587093110158862034399, kind CALL
[INFO ] 2020-01-02 17:31:2 <ether-query>&{1 [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] [16 156 79 44 204 130 196 215 123 222 21 243 6 112 115 32 41 74 234 63] 0xc0010cc6a0 [253 38 5 162 191 88 253 187 144 219 29 165 93 246 22 40 180 127 158 140] 0xc0010cc6e0 0xc000f6c3a0 CALL 6}
[INFO ] 2020-01-02 17:31:2 <ether-query>depth: 1, from 0x109C4f2CCc82C4d77Bde15F306707320294Aea3F, to 0x881b0A4e9c55d08e31d8d3C022144d75A454211c, transfer value 0, kind CALL
[INFO ] 2020-01-02 17:31:2 <ether-query>&{1 [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] [16 156 79 44 204 130 196 215 123 222 21 243 6 112 115 32 41 74 234 63] 0xc0010cc320 [136 27 10 78 156 85 208 142 49 216 211 192 34 20 77 117 164 84 33 28] 0xc0010cc360 0xc000f6c4e0 CALL 5}
[INFO ] 2020-01-02 17:31:2 <ether-query>depth: 1, from 0x109C4f2CCc82C4d77Bde15F306707320294Aea3F, to 0x834e9B529AC9Fa63B39A06f8D8C9B0D6791FA5dF, transfer value 0, kind CALL
[INFO ] 2020-01-02 17:31:2 <ether-query>&{1 [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] [16 156 79 44 204 130 196 215 123 222 21 243 6 112 115 32 41 74 234 63] 0xc000f6dfa0 [131 78 155 82 154 201 250 99 179 154 6 248 216 201 176 214 121 31 165 223] 0xc000f6dfe0 0xc000f6c4a0 CALL 4}
[INFO ] 2020-01-02 17:31:2 <ether-query>depth: 1, from 0x109C4f2CCc82C4d77Bde15F306707320294Aea3F, to 0xFD2605a2bF58fDbB90db1Da55dF61628B47F9e8c, transfer value 0, kind CALL
[INFO ] 2020-01-02 17:31:2 <ether-query>&{1 [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] [16 156 79 44 204 130 196 215 123 222 21 243 6 112 115 32 41 74 234 63] 0xc000f6d500 [253 38 5 162 191 88 253 187 144 219 29 165 93 246 22 40 180 127 158 140] 0xc000f6d540 0xc000f6c4c0 CALL 3}
[INFO ] 2020-01-02 17:31:2 <ether-query>depth: 1, from 0x109C4f2CCc82C4d77Bde15F306707320294Aea3F, to 0x881b0A4e9c55d08e31d8d3C022144d75A454211c, transfer value 6, kind CALL
[INFO ] 2020-01-02 17:31:2 <ether-query>&{1 [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] [16 156 79 44 204 130 196 215 123 222 21 243 6 112 115 32 41 74 234 63] 0xc000f6ca60 [136 27 10 78 156 85 208 142 49 216 211 192 34 20 77 117 164 84 33 28] 0xc000f6caa0 0xc000f6c800 CALL 2}
[INFO ] 2020-01-02 17:31:2 <ether-query>depth: 0, from 0x881b0A4e9c55d08e31d8d3C022144d75A454211c, to 0x109C4f2CCc82C4d77Bde15F306707320294Aea3F, transfer value 1000000000000000000, kind CALL
[INFO ] 2020-01-02 17:31:2 <ether-query>&{0 [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] [136 27 10 78 156 85 208 142 49 216 211 192 34 20 77 117 164 84 33 28] 0xc00028df00 [16 156 79 44 204 130 196 215 123 222 21 243 6 112 115 32 41 74 234 63] 0xc00028df40 0xc000487940 CALL 1}
[ERROR] 2020-01-02 17:31:2 <ether-query>calculateTransferValue fail! beforeBalance:2047918083944360637 currentBalance:3297918083944360629, block Number is: 50295
```

**对于函数运行过程中需要返回出来的值，对于指针而言，需要new一个新的对象，才能返回出来。否则值将会变成随机值**。

```golang
package main

import (
	"fmt"
	"unsafe"
)

type AA struct {
	a int
	b int
}

type BB struct {
	a int
	b int
}

func main() {
	var tmppp []BB
	var ttmp []*AA
	var tttmp []*AA
	tmp1 := BB{1, 2}
	tmp2 := BB{2, 3}
	tmppp = append(tmppp, tmp1, tmp2)
	tmp := tmppp
	// tmp3 相当于是一个临时变量，因此每次append的时候为tmp3
	// tmp3 为传值，不是传引用
	for i, tmp3 := range tmp {
		fmt.Printf("%v, %v\n", tmp3.a, ((*AA)(unsafe.Pointer(&tmp3))).a)
		fmt.Printf("%p, %p, %p\n", &tmp3, (*AA)(unsafe.Pointer(&tmp3)), &tmp[i])
		ttmp = append(ttmp, (*AA)(unsafe.Pointer(&tmp3)))
		tttmp = append(tttmp, (*AA)(unsafe.Pointer(&tmp[i])))
	}

	fmt.Printf("%v\n", ttmp)

	for _, tmp3 := range ttmp {
		fmt.Printf("%v\n", tmp3.a)
		fmt.Printf("%p\n", tmp3)
	}

	fmt.Printf("***************\n")

	fmt.Printf("%v\n", tttmp)

	for _, tmp3 := range tttmp {
		fmt.Printf("%v\n", tmp3.a)
		fmt.Printf("%p\n", tmp3)
	}
}

```

3. reverTransfer 中的revertNum初始值应该为-1， 避免transfer全部revert掉之后初始值0，还剩余一个transfer。

4. selfdestruct中转账的源地址为合约地址，目的地址为stack.pop()出来的地址，注意stack.pop()动作不要为了获取目的地址而执行两遍。

5. selfdestruct 

   该交易的stack为空

   123191

   0xbb8ee9866ee67277986b6f40775469c7a674810ce99dce3caff0d1117c8dcdac
   
   ```golang
   		if sLen := stack.len(); sLen < operation.minStack {
   			fmt.Printf("stack underflow (%d <=> %d)", sLen, operation.minStack)
   			return nil, fmt.Errorf("stack underflow (%d <=> %d)", sLen, operation.minStack)
   		} else if sLen > operation.maxStack {
   			fmt.Printf("stack limit reached %d (%d)", sLen, operation.maxStack)
   			return nil, fmt.Errorf("stack limit reached %d (%d)", sLen, operation.maxStack)
		}
   ```
   

6. 上面这笔交易是selfdestruct的错误，但是该错误的traces没有被抓取出来，原因是geth-query工具对于失败的交易不抓取traces，这显然不合理。改正：对于执行正确和错误(receipts.status)的交易都应该抓取traces。
7. 上面该交易有一个合约创建过程，并且这个合约创建过程不是由于CREATE trace导致的，而是最外面的create过程，最外层的call和create动作是不被记录在traces中的。并且还有一个问题：该交易首先create一个合约，然后该合约在构造函数中运行selfdestruct，出现`stack underflow (%d <=> %d)`错误，导致create的transfer交易被回滚，也就是所抓取的transfer交易都是正确执行的交易，错误以及被回滚的交易不会被记录在transfer表中。

8. 对于用户的balance值，postgres中不能设置成bigint，-9223372036854775808 到9223372036854775807，然而有值19900000000000000000。

   len(str(2**256-1)) = 78，使用类型numeric(78,0)
   