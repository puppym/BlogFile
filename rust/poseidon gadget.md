# poseidon gadget

## poseidon hash

为了在正确的场景使用persidon哈希，以下是一些注意点：

1. poseidon哈希所使用的有限域field通常由零知识证明系统所决定。大多数情况下他们是一条椭圆曲线的一阶循环子群上的一组点，椭圆曲线一般是BLS12-381, BN254, or Ed25519。poseidon将F长度的序列映射到长度为F的固定数组中。
2. 被哈希的消息为一个固定的长度或者是可变的长度(比如merkletree，总是2个节点的哈希值被哈希)
3. 防止哈希碰撞和原象攻击的安全等级为M。(大多数情况下为128bit)

有了上面的信息就可以确定poseidon排列的宽度w(以F元素的数量为单位):

1. 为容量保留C个元素，因此C个元素的F包含2M或者更多bit位。
2. 如果消息l的长度固定并且是非常小(10或者更小)，设置w=c+1。

然后你找出哪一个S-box是与曲线兼容的，对于曲线BLS12-381,BN254,Ed25519 S-box的最优值是x^5。

## poseidon hash 函数分类

在下面一共提出了两种hash函数：

STARKAD-Hash 是`binary case`情况下的哈希函数，该哈希函数是通过使用STARKAD-Permutation实例化一个`sponge`结构来进行构造。

POSEIDON-Hash 函数是`prime case`情况下的哈希函数，该哈希函数使用POSEIDON-Permuation实例化一个`sponge`结构来进行构造。



## poseidon hash 过程

poseidon hash  的代码一共可以分为4个部分，其中hash部分为多轮的操作，每轮操作分为3个部分：

1. paramters 该参数部分为论文给定的固定参数，一般情况下直接使用即可。
2. Add-Round Key，通过ARK(.)表示。
3. Sub Words Operation，通过S-Box(.)表示。
4. MixLayer，通过M(.)表示。

### poseidon hash 参数

```
def poseidon_params(p, t, nRoundsF, nRoundsP, seed, e, constants_C=None, constants_M=None, security_target=None):
    # Fq is the base field of Jubjub
	p:SNARK_SCALAR_FIELD = 21888242871839275222246405745257275088548364400416034343698204186575808495617 根据P值来计算n值，n值应该要满足安全目标(security_target)
    t:
    nRoundsF: full round 的轮数
    nRoundsP: partial round 的轮数
    seed: 为一个字符串的seed，通过blake2b哈希来哈希seed，从而取得随机数
    e: params.e 为 pow()的次方
    constants_C: 组合input的头部，加在input的前面
    constants_M: 在poseidon_mix中取M中的元素计算得出结果list
    
```



## poseidon hash 约束总结

### poseidon gadget

```c++
template<unsigned param_t, unsigned param_c, unsigned param_F, unsigned param_P, unsigned nInputs, unsigned nOutputs, bool constrainOutputs=true>
class Poseidon_gadget_T : public GadgetT
{
protected:
```

该gadget为poseidon的入口gadget。

**constraint**

constraint约束主要为各个round的约束。round主要分为FirstRoundT,PartialRoundT,FullRoundT,LastRoundT四种round

**withness**

分别为上述四类Round的withness。

### poseidon_Round gadget

为poseidon hash 每一轮的操作的约束过程

```c++
template<unsigned param_t, unsigned nSBox, unsigned nInputs, unsigned nOutputs>
class Poseidon_Round : public GadgetT {
public:	
	const std::vector<FifthPower_gadget> sboxes;
```

**constraint**

sboxes为一个FifthPower gadget的数组，为每个sboxes的约束过程就是该数组中每个FP gadget的约束过程。

**withness**

为sboxes数组中每一个FP gadget的withness过程。



### FifthPower gadget

为sBox数组中的元素。

**constraint**

一共有3个约束:

$x * x = x^2$

$x^2 * x^2 = x^4$

$x * x^4 = x^5$

```c++
	void generate_r1cs_constraints(const linear_combination<FieldT>& x) const
	{
		pb.add_r1cs_constraint(ConstraintT(x, x, x2), ".x^2 = x * x");
		pb.add_r1cs_constraint(ConstraintT(x2, x2, x4), ".x^4 = x2 * x2");
		pb.add_r1cs_constraint(ConstraintT(x, x4, x5), ".x^5 = x * x4");
	}
```

**withness**

对约束的中间变量x2，x4，x5进行赋值。

### 总结

因此poseidon hash的所有约束都集中在FifthPower Gadget的约束之上。



