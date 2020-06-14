

# 零知识证明zk-sNARKs实践

> 本问主要从实践角度来讲解`zk-sNARKs`，主要使用的`libsnark`库来通过代码完成一些例子。主要分为4个部分：
>
> `chapter1`：主要讲解从`欲证明问题` ->`circuit`->`R1CS(rank 1 constraint system)`->`QAP`->然后在QAP的基础上实现`zk-sNARKs`算法。
>
> `chapter2`：主要是通过结合`zcash`案例，实现一个简单链上隐私交易的例子。
>
> `chapter3`：主要是通过`loopring`案例，实现一个简单的链下交易验证的例子。
>
> `chapter4`：主要通过`filecoin`案例，实现一个简单的存储验证的例子。
>
> `chapter5`：主要通过`coda`案例，实现一个简单的区块压缩的例子。
>
> 上面的工作都是结合理论来解释对应的代码。

[TOC]

## zk-sNARKs 初探

本章主要讲解整个`zk-sNARKs`的工作流程和对其进行概述。

### zkp概述

### libsnark库安装

### libsnark库的用法

#### 欲证明问题到QAP的转化

#### 使用QAP结果结合libsnark组件实现zk-sNARKs

#### 在本地进行验证

#### 在以太坊上进行验证

### 结语





