2.21.

--朋友圈算法题--

测试昨天写的代码，并且整理文档，后面尝试和芦苇沟通，看看面板的数据展示

flex实验报告文档

2.22

沟通确定小论文投稿方向

弄grafanna面板的sql语句查询。

算法题2道

2.29

下午补充飞书上面的项目文档

晚上继续修改小论文内容，使用文献管理器管理参考文献。

1. 未来干什么？时间有限
2. 最有价值，做别人替代不了的。

3.1

按照想法修改小论文。

修改geth-query代码，测试。

跑merkletree电路。



3.6 

1. leetcode每日一题
2. 面经复习
3. sm2/3算法测试
4. 开题报告修改，上传。



3.7

1. leetcode
2. 面经 或者把loopring的总结一下。
3. 国密算法



3.9

1. 准备腾讯面试相关：主要在以下方面，print的编码，golang，一些基础的算法过一遍，然后是剑指offer过一遍。（上午）
2. 看第10章trait，然后拉下SM的代码，跑SM2和看其实现原理。对照着学姐的文档边看边总结。
3. 晚上，看今天啥时候面试，主要是看rust的SM2实现。



3.24

1. 总结bellman的gadgets和circuit约束系统的实现。
2. 继续研究rust的trait一章。
3. leetcode一道题目。





1. zkp0
2. zkp1
3. zkp2
4. zkp3
5. zkp4
6. stateDB
7. geth-query博客整理
8. **import_check_createIndex.py**   报错信息显示终止。
9. **import_realtime_data_each_1000.py** blocks导入报错信息显示。并且终止程序
10. 实时区块数据中间表导入方案。

$$result = \sum  bits[i] * 2^i$$





**4.20** 

1. 逗号和顿号
2. 两个图片保留右边的图，工具的效率分析图，曲线上的点要去掉。
3. 数据种类丰富度需要进一步增加分析的工具种类。
4. 融合第三部分的背景知识到前两个部分
5. 重新作图
6. 导出的数据种类分类并且作图分析
7. 1-7个步骤的排版格式
8. 摘要里面这些数据不仅仅是为了合约的安全性分析。



我为什么需要这些数据(背景)

我需要哪些数据

怎么可以获得

我是怎么获得这些数据

测试结果

$f(x) = a_0 + \sum_{i=1}^{k-1} a_i x^i$

$f(x) = a_0 + \sum_{i=1}^{k-1} a_i x^i$

- 最后的分析图重新作
- 需要增加导出数据工具的分析。
- 





1. 和学姐共同梳理了bellman gadget开发文档。
2. 整理了libsnark和ethsnark的大部分gadget，并且写了一些文档。
3. 把以前loopring的电路总结文档上传。
4. 请假了2天。



下周工作：

1. 完成merkletree gadget。
2. 后续一些简单的gadget的编写。



1. 小论文的修改。修改了部分语句和图内容。
2. 过了一遍poseidon hash的论文，很多地方还没看明白。
3. 大致梳理了ethsnark中poseidonhash gadget的代码模块，在看的过程中发现zencash库是里面有rust 版本 posidon hash gadget的实现，正在研究代码。



下一步准备先看明白ethsnark，poseidon的python版本的实现，然后再考虑poseidon gadget的写法。



论文改写工作

1. 提出问题，分析问题，解决问题。一般提出问题和分析问题占摘要的一段，解决问题占摘要的一段，并且解决问题过程中需要初略的进行结果对比，突出结果的优势。(摘要里面明确论文结果的优势)
2. 论文的两个主要的创新点在于1. 链上数据非常有价值(合约安全分析是一个方面)，需要导出详细的数据，但是一般的工具导出数据丰富度不够，该工具导出数据种类丰富。2. 该工具导出速度快，于dataEth工具相比，该工具速度提升较大。
3. 介绍部分的语言组织就是将摘要中的语句进行扩充。





5.15 最近两周的计划如下：

1. 基本完成ethsnark和loopring中一些基本gadget的编写工作。
2. bellman工作的总结。
3. 零知识证明(NIZK)总结。
4. 实习资料汇总。
5. 完成小论文的初稿以及投递。
6. 完成入职相关事宜(入职体检，入职资料准备，租房事宜)。
7. 每天坚持一道leetcode。
8. 去南京玩一圈。



5.19号

总结:

理不清楚poseidon hash 逻辑在gadget中的逻辑理不清楚。

上周准备吧poseidon hash gadget的实现先放一放，准备优先实现ethsnark中的gadget，上周已经实现了部分的gadget，isnonzero，Lookup1bit，Lookup2bit，Lookup3bit。这周准备实现ethsnark和loopring中剩余的gadget。

1. 完成Lookup系列的电路。
2. commit 电路代码。
3. 上传poseidon hash 的文档。



5.25

上周总结：

完成了大部分ethsnark中gadget文件夹之下除去hash之外的大部分gadget，并且写的代码commit到了zkp-toolkit库的gadget文件夹下面。这周准备开始移植loopring中的常见gadget的移植。



5.30 最近两天的计划

1. 整理一下go-ethereum 的代码。



