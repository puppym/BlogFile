* [ ] ## 12.2

  重入攻击交易分析

  smartbug库中的重入合约的例子，在以太坊主网中没有能够找到调用这些合约的交易。只有这个合约地址有交易调用的记录，当时该记录没有明显的重入合约的调用特征。

  ```
  0x941d225236464a25eb18076df7da6a91d0f95e9e
  ```

  

  ## 12.3

  - [x] withdraw电路
  - [x] min电路
  - [x] 区块链课程实验检查统计，发给燕姐

  

  ## 12.4 

  - [x] updateAccount 电路
  - [x] 区块链课程实验检查统计，发给燕姐

  

  ## 12.5

  - [x] transfer电路

  - [x] leetcode LRU 缓存

    transfer电路还有一些问题，明天调试通过

    

    ## 12.6 

    - [x] leetcode LRU缓存
    - [x] 实验统计
    - [x] 浙江省接收函办理
    - [x] leetcode random list
    - [x] transfer 电路调试

    

## 12.7 

* [ ]  leetcode random list 

## 12.8

* [ ] 以太坊数据导出工作总结，准备明天的交接
* [ ] 计算机科学杂志文稿修改



## 12.9

* [x] 大论文讨论。
* [x] 小论文工作修改。
* [ ] 相关资料整理发布到群里
* [ ] 整理

最近的工作：

* [ ]  导出22个账户的所有攻击交易，将攻击交易的指定字段导出成csv数据。
* [ ]  写一个攻击交易环检测的脚本，尝试统计所有攻击交易存在的环的个数。
* [ ]  导出DAO攻击最近的100W个区块的所有交易的trace数据，尝试找出更多可能的攻击账户和攻击交易。



* [ ] 找合约源代码的数据集，找合约字节码的数据集。

 

## 12.10

* [ ] 导出重入数据到csv文件
* [ ] 编写csv文件数据分析脚本，

## 12.10

* [ ] 汇总22个账户的所有攻击交易。
* [ ] 导出DAO附近的50W个区块数据，并且分析出其它可能是DAO攻击的地址(脚本)。
* [ ] 汇总以上数据



## 12.11

## 12.12

1. GethQuery有一个bug，当一笔交易失败，可能在调用栈的第5层失败，前4层的trace都被记录，但是通过call指令捕获的所有transfer数据都被revert掉了，存在的bug是transfer数据的revert函数写的有问题。



2. transaction的错误原因压根就没有填进transaction的数据当中。

`data.transactions = append(data.transactions, transtrace)` 



## 12.13 

10个环的结果
DataEther
0x1EB9BD9C2236649B15EE8BE1961B40397A64A166  0xfa19dcc4af83627730f63ca92281a87d00e3c5d9f06b173d55e2ce5a47283440 有3个环
0x4613F3BCA5C44EA06337A9E439FBC6D42E501D0A  这个账户是从etherscan中是没有重入交易的(可以在etherscan中进行查看)
0x900a979CFCC4a9e5F0DCaC1f7cc629873e2528Ec  0x4e24cce9643ec2d60175485f0d39642ffbda20383a550c63d6679607a634958c 有一个环

my_address
0x34a5451ef61a567ee088dcf5f324bfbc4bcf426f 新发现的重入攻击地址
0xe306aac52823ba1d3938608381a2444d9d641cc1 新发现的重入攻击地址：0x7afa6823e7f53a96fc7fedbee7145e1654252bfb7544a8c0a8ae825507ae1203

重新跑一遍脚本，将环的数据设置为1即统计为重入攻击交易，地址为重入攻击地址。

0x4613F3BCA5C44EA06337A9E439FBC6D42E501D0A  感觉https://ethereum.stackexchange.com/questions/6320/how-many-the-dao-recursive-call-vulnerability-attacks-have-occurred-to-date该链接中的9个地址该地址不是其中的地址，可能统计错误。



## 12. 14

* [x] 找到# 1 The major 17 June 2016 attack #1 that pictured in [ether.camp/dao-thief](http://www.ether.camp/dao-thief.html) (info from user [iamtrillion](https://forum.daohub.org/users/iamtrillion) in the post [DAOhub.org - [Workgroup\] DAO White Hat Team](https://forum.daohub.org/t/workgroup-dao-white-hat-team/5355/25?u=bokkypoobah)). 2016.6.17最初的攻击交易的信息。

  这里的最初信息就是最初统计的2个地址，也就是在该链接中只有统计8个攻击地址

* [x] 列出毕业论文大纲

* [ ] leetcode 2题

## 12.15 

* [ ] 完成merkletree_sha256bug 修复。
* [ ] 数据统计脚本修改运行，准备导出数据集。
  1. 全部的数据集合
  2. 全部数据的重入攻击交易数据集合
  3. 22个账户所有交易的数据集合
  4. 22个账户所有重入交易的数据集合。

## 12.16 

* [ ] 写一天毕业论文



## 12.17 

* [ ] 写一天毕业论文
* [ ] 完成了毕业论文的第一章，对于国内外的研究方面还有待加强。

http://a1.qpic.cn/psc?/V10Wk5As28o59d/ruAMsa53pVQWN7FLK88i5t3Rsq.0EKT8HDcan*PnMackTvsZgznouWu*vmA.Z8j5aqr7K51Eg9eqzXNa1wpUP343GqMDDqrjooFhvIpMGK0!/b&ek=1&kp=1&pt=0&bo=QAZVCEAGVQgRECc!&tl=3&vuin=1850226700&tm=1608116400&sce=60-4-3&rf=viewer_311

## 12.18

* [ ] 回公安

* [ ] 计划下一阶段的工作

  回公安，需要调整接下来的工作。



## 12.19

* [ ] 整理重入攻击的数据集，并且将数据集导出。
* [ ] scp -P 10505 czm@101.132.106.50:/home/czm/reentrancy_all_data_160W_220W.zip .
* [ ] leetcode 

## 12.20 

* [ ] 毕业论文



## 12.21 

* [ ] 毕业论文
* [ ] 重入攻击数据集整理

## 12.22 

* [ ] 毕业论文摸鱼



## 12.23

* [ ] 毕业论文摸鱼
* [ ] leetcode2道
* [ ] 区块链底层技术与研究概述

## 12.24

* [x] 区块链底层技术与研究概述
* [ ] clover-protocol 数字签名部分
* [x] leetcode

## 12.25 

* [ ] 云南旅游计划
* [ ] leetcode 2道
* [ ] 数字签名部分整理工作量
* [ ] 区块链相关文档

## 12月份完成的任务

1. 毕业论文初稿。
2. clover-protocol的签名验证部分的代码。
3. 每天2道高频leetcode



12月份的任务没有完成

原因：

1. 自己偷懒。
2. 没有在空闲时间中合理安排好自己的时间。
3. 由于自己的生活进行不足，造成一些不必要的烦恼。

改进：

1. 自己提高自身的学习积极性
2. 合理安排自己的学习时间。