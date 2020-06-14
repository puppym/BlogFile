### 问题列表

1. RingSettlementCircuit.h`电路中的`TransformRingSettlementDataGadget`的`generate_r1cs_constraints`函数能够进一步优化为以下代码。

```c
struct Range
{
    unsigned int offset;
    unsigned int length;
};
std::vector<Range> ranges;
ranges.push_back({0, 40});   // orderA.orderID + orderB.orderID
ranges.push_back({40, 40});  // orderA.accountID + orderB.accountID
ranges.push_back({80, 16});  // orderA.tokenS + orderB.tokenS
ranges.push_back({96, 48});  // orderA.fillS + orderB.fillS
ranges.push_back({144, 8});  // orderA.data
ranges.push_back({152, 8});  // orderB.data
for(const Range& subRange : ranges)
{
    for (unsigned int i = 0; i < numRings; i++)
    {
        transformedData.add(subArray(data, i * ringSize + subRange.offset, subRange.length));
    }
}
```

2. MatchingGadget中注释写错。`        takerFillB_lt_makerFillS(pb, takerFill.B, makerFill.S, NUM_BITS_AMOUNT, FMT(prefix, ".takerFill.B < makerFill.B")),`应该为`".takerFill.B < makerFill.S"`
3. 对于main.cpp中的`prove`和`generatekeys`两个动作同时进行可能存在问题。有两种逻辑：首先pk和vk不存在，在生成证明的时候首先generatorkeys，然后再proof，这种情况逻辑上说得通，但是不符合常理。当pk和vk不存在时，不允许执行prove操作。因为generatorkeys是trust setup，因此这一步骤需要多人参与并且只产生一次。因此这里更好采取后面一种使用方法。
4. 对于Data.h文件中，offchainWithdrawBlock类中，`startIndex`和`startHash`没有使用，貌似直接从`onchainWithdrawBlock`中复制过来😂。
5. 关于`MatchingGadgets`这个组件分析

|          caseC          | Taker     | Maker     |
| :---------------------: | --------- | --------- |
|         tokenS          | GTO       | WETH      |
|         tokenB          | WETH      | GTO       |
|         amountS         | 101       | 101       |
|         amountB         | 100       | 100       |
|          isBuy          | Yes       | no        |
|         fill.S          | 99        | 100       |
|         fill.B          | 100       | 99        |
| target price (WETH/GTO) | 1.01      | 100/101   |
| settle price (WETH/GTO) | 0.99      | 0.99      |
|     balance tokenS      | 2         | 1         |
|     balance tokenB      | 100       | 99        |
|    order sale price     | 100/101 W | 100/101 G |
|      settle price       | 100/99 W  | 99/100 G  |

对于taker：`100/99 > 100/101`，因此成交卖出价格高于订单卖出价格。

对于maker: `100/101 > 99/100`因此实际卖出价格是低于订单卖出价格。但是其依旧是能够成交，依赖于`(fillAmountS * amountB * 100) < (fillAmountB * amountS * 101)`

总结：订单匹配有益于taker。其余情况参考。 [link](