## zksnarks 简介

零知识证明(zk-snark) 是

的缩写：

* zero knowledge:证明者在不泄露欲证明隐私
* Succint: 证明数据量比较小
* Non-interactive:没有或者很少有交互
* ARguments:计算可靠性证明，只是在有限计算量的情况下有效，拥有足够计算量的证明者可以伪造证明，也称为计算可靠性（相对的还有完美可靠性）
* of knowledge: 对于证明者来说在不知道证据的情况下(witness)，构造出一组参数和证明是不可能的。



零知识证明有四个部分构成，多项式问题的转化，随机挑选验证，同态隐藏，零知识。

* 多项式问题的转化－需要证明的问题转化为多项式问题`u(x) · v(x) = w(x) + h(x) · t(x)`，证明者提交证明要验证者验明多项式成立。
* 随机挑选验证，随机选择验证的数值s，验证 上面等式成立，随机挑选验证，简单并且验证的数据量少，随机挑选验证的安全性肯定没有全部验证的安全性高，但是如果足够随机，安全性也是相当高的。
* 同态隐藏，是函数的一种特性，输入的计算和输出的计算保持同态，以加法计算为例，满足如下三个条件的函数E(x)成为加法同态1. 给定E(x)，很难推出x(这条特性有群论基础，椭圆曲线)。2. 不同的输入对应不同的输出。3. E(x+y)可以由E(x)和E(y)计算出来，乘法同态类似。
* 零知识：证明者和验证者之间除了“问题证明与否”知识之外，不知道其他任何知识(不知道随机挑选值，不知道挑选值的多项式计算结果等等)。[zksnark入门1](https://mp.weixin.qq.com/s/M1ZAXPSPqO8KpE3VyExwTw)



### 欲证明问题转化为QAP过程

要使用zk-snark零知识证明，首先将要证明的东西展平，形成一个计算，并且将该计算用只包含加法和乘法的电路形式表达出来，然后将该计算电路转化为QAP(QAP Quadratic  Ari)。总的来说分为下面几步：

1. 首先将欲证明的问题转化为电路(circuit)。
2. 然后将电路转化为R1CS。
3. 然后将R1CS转化为QAP(Quadratic Arithmetic program)。
4. 最后基于QAP实现zksnark的证明算法。[zksnark入门2](https://mp.weixin.qq.com/s/M1ZAXPSPqO8KpE3VyExwTw)



了解QAP问题，首先了解NPC问题：

* P问题：是多项式时间内可解问题，与之对用的是NP问题。
* NP问题：是多项式时间内不可解的问题，但是能够给定一个解，在多项式的时间内验证这个解是否正确。
* NPC问题：是NP问题的核心，任何NP问题都可以归结为一个NPC问题，如果某NP问题的NPC问题解决了，那么也就意味着该问题的NP问题解决了。

QAP问题其实是一个NP问题，也就是说QAP问题可以在多项式时间内验证一个解是否正确，但是一个在有限时间内，使用有限的资源是很难在多项式时间内推出一个正确的解的。

QAP是这样的一个NP问题，给定一系列的多项式，以及给定一个目标多项式，找出多项式的组合能够整除目标多项式。

关于验证一个QAP问题的解的详细过程参考博客：[QAP的详细验证过程](https://mp.weixin.qq.com/s/M1ZAXPSPqO8KpE3VyExwTw)

### zksnark 概述

`C(X,W) == true` 

C 表示某一程序

X 表示公共输入

W 是私钥的输入

### 例子

`hash(s) == H`

Alice 拥有私钥s，Bob和Alice都明显知道公钥H，现在出现一种情况。Alice既想证明自己拥有公钥H的私钥，又不想将自己的私钥暴露给Bob，因此这个问题能够用zksnarks来解决。

转换问题可变得证明如下：

`C(H,s) == true`

### zksnarks的具体定义

主要分为3个步骤：

1. Generator:

$$
Generator(C circuit, \lambda ...)
\\
(pk, vk) = G(\lambda, C)
$$

密钥产生器G通过lambd和电路程序C，产生一个证明密钥pk和验证密钥vk。这两个密钥仅仅需要被产生一次。并且在产生后lambda即被摧毁。	

2. Prover:

$$
Prover(x \quad pub \quad input, w \quad sec \quad input)
\\
\pi = P(pk, x, w)
$$



这个证明机器P通过输入的证明密钥pk，公钥输入x和一个私钥的签名w，这个证明机器能够产生证明`prf = P(pk,x, w)`， 证明者证明w输入是满足这个程序的。

3. Verifier:

$$
V(vk, x, \pi) == (\exists w \quad s.t \quad C(x, w))
$$

验证器V计算`V(vk, x, prf)`， 返回true如果证明是正确的。因此这个函数返回true，如果证明器知道签名w满足`C(x,w) == true`。

### 回到上面的例子

1. Bob Generator。但是这里不能是Bob generator，因为不能保证Bob是否保存了lambda的参数。但是Bob和Alice和无穷多的人能够一起进行这一步，这样就没有一个人知道lambda。
2. Alice Prover。
3. Bob verifier。

### zksnarks在以太坊上的应用

验证算法以预编译合约的形式添加到以太坊中。Generator和Prover是脱链运行的，最后在链上进行证明，验证密钥和公共输入作为输入参数，在智能合约中运行通用的验证算法，来对证明进行验证。可以使用验证算法的结果来触发其他链上的活动。

### 参考文档

 [github介绍1](https://media.consensys.net/introduction-to-zksnarks-with-examples-3283b554fc3b)

[Vitalik](https://medium.com/@VitalikButerin/quadratic-arithmetic-programs-from-zero-to-hero-f6d558cea649)

[zksnark通俗解释](https://mp.weixin.qq.com/s/M1ZAXPSPqO8KpE3VyExwTw)

[零知识证明 - zkSNARK入门-带有少量公式](https://mp.weixin.qq.com/s/M1ZAXPSPqO8KpE3VyExwTw)