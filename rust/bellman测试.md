## bellman-exmaple

[TOC]



### 运行bellman测试

```
1. 去除Cargo.toml中的ff的路径，不知道是版本原因或者其它原因，这里加了path之后一直显示不能找到ff的路径。
2. cargo build
3. cargo test
这里会报错，修改行fn random<R: RngCore + ?std::marker::Sized>(rng: &mut R)
4. cargo test  test_mimc --  --test-threads 1 --nocapture 运行/tests/mimrc.rs中的测试用例, test_mimc的意思是只运行该测试用例，‘--’ 之后表示参数，--test-thread表示测试线程数目，--nocapture表示打印输入输出
```

### 动手根据写一个小例子

还是使用libsnark[教程](https://github.com/christianlundkvist/libsnark-tutorial) 的小例子`x^3+x+5 == 35`。(后面发现github上有用bellman实现的关于该小例子的证明。)

**rust代码总的步骤如下：**

1. `let rng = &mut thread_rng();`产生随机数rng。

2. 电路约束实现，对于电路约束实现主要有以下步骤：

   2.1. 定义问题的结构体，对于问题`x^3+x+5 = out`问题的结构体如下：

   ```rust
   pub struct CubeDemo<E: Engine> {
       pub x: Option<E::Fr>,
   }
   ```

   可以理解CubeDemo中的参数为withness生成prove所需要的参数个数。上述问题的witness只有一个参数x。

   2.2. 实现Cube问题结构体的电路，首先拍平原问题：

   ```rust
   x * x = sym_1
   sym_1 * x = y
   y + x = sym_2
   sym_2 + 5 = out
   
   通过引入witness = [one, x, out, sym_1, y, sym_2]向量将上述电路转化为R1CS的约束系统。然后通过拉格朗日插值将R1CS转换为QAP(二次多项式问题)。
   ```

   在生成电路的过程中主要有以下2个问题：

   * 生成变量。生成private "auxiliary" variable和public "primary" varibale的接口不同。对于private我的理解是和withness以及由witness通过计算得到的(x,sym_1,sym_2)。对于public是相当于`L(s)R(s)-O(s) = h(s)*t(s)`中的t(s)，该值相当于是一个公开的值。(相当于verifier将随机数带入到公开的多项式中得到的公开的值)。private变量的生成使用`let x = cs.alloc()`。public变量使用`cs.alloc_input()`。
   * 构建`constraint`约束`cs.enforce()`，一般的也将加法转换为乘法。

3. 为电路创建参数。通过随机值rng为电路创建CRS(公共引用参考串)。`generate_random_parameters(c, rng).unwrap()`。

4. 从proof key中生成verify key。`let pvk = prepare_verifying_key(&params.vk);`

5. 创建Cube电路实例，并且传入withness的x值。`let c = CubeDemo::<Bls12> {x: Fr::from_str("3")};`。

6. 用电路实例加上withness产生proof。`let proof = create_random_proof(c, &params, rng).unwrap();`

7. 最后assert进行验证proof。`verify_proof()`

### 具体代码实现

[bell-example例子代码实现](https://github.com/arcalinea/bellman-examples)

上面的例子中一共实现了两个问题`A*B=c`和`x^3+x+5 = out`。`multiply.rs`实现的是前者，`cube.rs`实现的是后者。不过两者的具体流程都和上述一样，只是通过电路构造的约束系统不相同。



### bellman 代码分析

bellman库分离了有限域大数运算`ff`，群组运算`group`，特定证明系统库`Groth16`，以及双线性配对库`pairing`，通过自定义拉取各类基础库可以达到切换不同曲线和不同证明系统实现零知识证明的效果。

#### bellman代码结构

```bash
bellman
├── COPYRIGHT
├── Cargo.lock
├── Cargo.toml    // 项目配置文件
├── LICENSE-APACHE
├── LICENSE-MIT
├── README.md
├── src
│   ├── domain.rs
│   ├── gadgets    // 各个gadget的实现
│   ├── gadgets.rs // 各类gadget接口的暴露
│   ├── groth16    // groth16证明系统的实现
│   ├── lib.rs     // bellman向外暴露的接口
│   ├── multicore.rs // bellman并且计算的接口
│   └── multiexp.rs
├── tests
│   └── mimc.rs    // bellman的一个mimc哈希验证测试
└── ~
```

#### gadgets写法

**/src/gadgets**

gadget是ConstraintSystem上面的一个一个组件，通过多个gadget之间输入输出的相互组合形成整个约束系统。

下面主要分析`boolean.rs`中的一些gadgets，这些gadgets相当于在电路中运行bool逻辑，首先下面结构体表示在约束电路中的变量，分为值和电路变量表示。其`value`值为调用gadgets传入的数值，`Variable`为`cs.alloc`分配的变量。

```rust
#[derive(Clone)]
pub struct AllocatedBit {
    variable: Variable,
    value: Option<bool>,
}
```

下面分析各个gadget的作用：

* `alloc_conditionally` 如果value为none，结果报错。如果value为true，b为false ，结果为sucesee。如果value为true，b为true，结果为fail。

  总体约束为` Constrain: (1 - must_be_false - a) * a = 0`

  如果b为true，暗示着value = 0;

  如果b为false，a为任意值

* `alloc` `Constrain: (1 - a) * a = 0` 限制a为0或者1

* `xor`  `Constrain (a + a) * (b) = (a + b - c)`异或运算（满足异或运算的3个操作符应该满足该等式）。

* `and` `Constrain (a) * (b) = (c)`

* `and_not` `Constrain (a) * (1 - b) = (c)` Calculates `a AND (NOT b)`

* `nor` `Constrain (1 - a) * (1 - b) = (c)`   Calculates `(NOT a) AND (NOT b)`

**下面针对xor运算符分析其具体写法**

```rust
 /// Performs an XOR operation over the two operands, returning
    /// an `AllocatedBit`.
    // 三个参数分别为CS约束系统，该gadget上的两个输入参数，Result<Self, SynthesisError>为返回值，可以用Ok宏来包裹返回值
    pub fn xor<E, CS>(mut cs: CS, a: &Self, b: &Self) -> Result<Self, SynthesisError>
// where表约束
    where
        E: ScalarEngine,
        CS: ConstraintSystem<E>,
    {
        let mut result_value = None;
        // 分配C值，并且C的值成员变量为a^b的值
        let result_var = cs.alloc(
            || "xor result",
            || {
                if *a.value.get()? ^ *b.value.get()? {
                    result_value = Some(true);

                    Ok(E::Fr::one())
                } else {
                    result_value = Some(false);

                    Ok(E::Fr::zero())
                }
            },
        )?;
        // 将a,b,c三者之间的关系添加到约束系统CS中
        // Constrain (a + a) * (b) = (a + b - c)
        // Given that a and b are boolean constrained, if they
        // are equal, the only solution for c is 0, and if they
        // are different, the only solution for c is 1.
        //
        // ¬(a ∧ b) ∧ ¬(¬a ∧ ¬b) = c
        // (1 - (a * b)) * (1 - ((1 - a) * (1 - b))) = c
        // (1 - ab) * (1 - (1 - a - b + ab)) = c
        // (1 - ab) * (a + b - ab) = c
        // a + b - ab - (a^2)b - (b^2)a + (a^2)(b^2) = c
        // a + b - ab - ab - ab + ab = c
        // a + b - 2ab = c
        // -2a * b = c - a - b
        // 2a * b = a + b - c
        // (a + a) * b = a + b - c
        cs.enforce(
            || "xor constraint",
            |lc| lc + a.variable + a.variable,
            |lc| lc + b.variable,
            |lc| lc + a.variable + b.variable - result_var,
        );
        // 返回AllocatedBit的c值对象
        Ok(AllocatedBit {
            variable: result_var,
            value: result_value,
        })
    }
```

具体步骤如下：

1. 定义result_value和result_var。分别计算两者的值，result_value的值和约束的不同计算过程类似。result_var的值需要`cs.alloc`函数进行分配。
2. 根据输入和输出变量之间的关系，构造约束等式`(a + a) * (b) = (a + b - c)`，该过程可以通过`¬(a ∧ b) ∧ ¬(¬a ∧ ¬b) = c`推的。
3. 根据约束等式通过`cs.enforce`函数将约束等式添加到CS约束系统中。
4. 返回`AllocatedBit`对象。

#### bellman gadget 功能

```bash
gadgets
├── blake2s.rs   // blake2s哈希实现
├── boolean.rs   // 一些bool运算gadgets的实现
├── lookup.rs    // 窗口表查找gadget(Window table lookup gadgets)，按照bit位查找窗口表的值

├── multieq.rs   // MultiEq子模块该子模块有自己的成员函数也实现了CS trait中的函数。
├── multipack.rs // 将字节数组转化为标量元素
├── num.rs       // 表示基础曲线标量字段的小工具
├── sha256.rs    // sha256哈希函数电路
├── test
│   └── mod.rs   // 测试电路子模块TestConstraintSystem系统重新实现CS trait
└── uint32.rs     //uint32约束子模块，帮助sha256哈希的计算
```



#### ConstraintSystem原理

**/src/lib.rs**

首先申明一个ContraintSystem的trait里面定义了构建约束系统的方法。

`pub trait ConstraintSystem<E: ScalarEngine>: Sized`

对于约束系统而言一共有以下函数:

- `fn one() -> Variable`返回约束系统中的常量`CS::one()`。
- `fn alloc` 在CS上分配一个私有变量（withness）。
- `fn alloc_input`在CS上分配一个共有变量。
- `fn enforce`在CS上乘法约束组合两个输入和一个输出变量。
- `fn push_namespace`和`fn pop_namespace`仅仅只是`root constraint system`用户能够调用该函数？
- `fn get_root`返回根约束对象



然后使用一个`namespaced`的约束系统对上述约束系统的实现

`impl<'cs, E: ScalarEngine, CS: ConstraintSystem<E>> ConstraintSystem<E> for Namespace<'cs, E, CS>`

`impl<'cs, E: ScalarEngine, CS: ConstraintSystem<E>> ConstraintSystem<E> for &'cs mut CS`

namespace的概念就是生成一个分成结构，把一个电路中的多个gadget模块划分成多个子模块，对于每个namesapce的对象也要实现ConstraintSystem的trait。

#### range proof

##### libsnark comparsion_gadget分析

![img](https://pic1.zhimg.com/80/v2-9cc8cd683750afbf82c559e591f098b8_720w.jpg)

comparsion_gadget的构造函数主要是：给定两个n位的数A和B，输出两个变量：less和less_or_eq(A是否小于或等于)。构造函数初始化其它的私有成员变量的意思为：

`pb_variable_array<FieldT> alpha`:alpha是长度为n+1的变量数组，其中alpha[n]是less or eq。alpha是2^n+B-A的结果的位的表示

`pb_variable<FieldT> alpha_packed`:alpha_packed是一个变量，alpha对应的值。也就是，alpha_packed等于2n+B-A

`std::shared_ptr<packing_gadget<FieldT> > pack_alpha`:pack_alpha是packing_gadget，将n+1位的alpha打包，计算结果存储在alpha_packed变量中。packing_gadget就是将位表示的数据，变成数值表示。

`std::shared_ptr<disjunction_gadget<FieldT> > all_zeros_test`:all_zeros_test是disjunction_gadget，确定alpha的n个变量中是否全0。

`pb_variable<FieldT> not_all_zeros`:not_all_zeros是一个变量，表示B-A的结果是否全是0。

**generate_r1cs_constraints**函数分析

``` c++ 
template<typename FieldT>
void comparison_gadget<FieldT>::generate_r1cs_constraints()
{
    /*
    // 该值对应alpha_packed的值
      packed(alpha) = 2^n + B - A
      // 将0 --- n-1取或 
      not_all_zeros = \bigvee_{i=0}^{n-1} alpha_i
      
      // alpha[n] 相当于符号位
      if B - A > 0, then 2^n + B - A > 2^n,
          so alpha_n = 1 and not_all_zeros = 1 and less_or_eq = 1
      if B - A = 0, then 2^n + B - A = 2^n,
          so alpha_n = 1 and not_all_zeros = 0 and less_or_eq = 1
      if B - A < 0, then 2^n + B - A \in {0, 1, \ldots, 2^n-1},
          so alpha_n = 0 and not_all_zeros = 1 and  less_or_eq = 0
          
      // 由上面的枚举可以得知当B >= A时 alpha[n]=1, 如果not_all_zeros的值为1，则B>A,否则B=A
      therefore alpha_n = less_or_eq and alpha_n * not_all_zeros = less
     */
    // 1. 对not_all_zeros，添加boolean约束（该变量只能是0或者1）。将not_all_zeros转换为一个Boolean，packing_gadgets中的所有位都是一个Boolean。
    /* not_all_zeros to be Boolean, alpha_i are Boolean by packing gadget */
    generate_boolean_r1cs_constraint<FieldT>(this->pb, not_all_zeros,
                                     FMT(this->annotation_prefix, " not_all_zeros"));
    
    // 2. pack_alpha->generate_r1cs_constraints(true) 约束alpha对应的数值等于alpha_packed。将n+1位的alpha打包，计算结果存储在alpha_packed变量中。packing_gadget就是将位表示的数据，变成数值表示。
    /* constraints for packed(alpha) = 2^n + B - A */
    pack_alpha->generate_r1cs_constraints(true);
    
    // 3. 1*(2^n+B-A) = alpah_packed
    this->pb.add_r1cs_constraint(r1cs_constraint<FieldT>(1, (FieldT(2)^n) + B - A, alpha_packed), FMT(this->annotation_prefix, " main_constraint"));
    
    // 给结果添加约束
    /* compute result */
    // 4. 确定not_all_zeros变量的值和alpha中n个变量中是否为0的结果一致
    all_zeros_test->generate_r1cs_constraints();
    // 5. less_or_eq * not_all_zeros = less
    this->pb.add_r1cs_constraint(r1cs_constraint<FieldT>(less_or_eq, not_all_zeros, less),
                                 FMT(this->annotation_prefix, " less"));
}
```



#### MinGadget依赖关系

![comparsion_gadget](C:\Users\czm18\Desktop\备份\blog\rust\comparsion_gadget.png)



#### bellman和libsnark中变量的结构体

##### R1CS

R1CS(rank one constraint system)，$A*B=C$ 其中ABC的类型都为`linear_combination`类型。众多的R1CS约束组成了一个约束系统。`trust setup`步骤生成pk和vk过程是依赖于整个的约束系统。生成证明的过程就是将withness S带入约束系统$S*(A*B) = S*C$加上pk生成proof的过程，注意这里的withness S是包含private input 以及 public input 。验证proof的过程就是取一个随机值验证$S*(A*B) = S*C$等式是否成立，也就是通过proof加上vk来验证输入的withness S是否是正确的秘密。

R1CS是零知识证明计算的表达方式，任何基本的运算都能拍平成R1CS的形式，一个关于向量W的约束被定义成以下形式：

`<A, W> * <B, W> = <C, W>`

其中A， B， C是与W相同长度的向量，<> 代表向量的内积，下面代表R1CS的向量方程组：

```bash
<A_1, w> * <B_1,w> = <C_1, w>
<A_2, w> * <B_2,w> = <C_2, w>
...
<A_n, w> * <B_n,w> = <C_n, w>
```

向量W在zksnark中被称为withness(秘密)。由zksnark算法生成的proof总是能够证明证明者知道能够使得上面R1CS向量组满足的w向量的值，也就是证明证明者在生成proof的过程中是知道并且加入了自己的withness秘密。



##### FieldT 和 E::Fr

为有限域上的值。该值依赖于具体的椭圆曲线。

**libsnark**

```c++
/**
 * Gadget that represents an Fp4 variable.
 */
template<typename Fp4T>
class Fp4_variable : public gadget<typename Fp4T::my_Fp> {
public:
    typedef typename Fp4T::my_Fp FieldT;
    typedef typename Fp4T::my_Fpe Fp2T;
```

**bellman**

```rust
/// An "engine" is a collection of types (fields, elliptic curve groups, etc.)
/// with well-defined relationships. In particular, the G1/G2 curve groups are
/// of prime order `r`, and are equipped with a bilinear pairing function.
pub trait Engine: ScalarEngine {
    /// The projective representation of an element in G1.
    type G1: CurveProjective<
            Engine = Self,
            Base = Self::Fq,
            Scalar = Self::Fr,
            Affine = Self::G1Affine,
        > + From<Self::G1Affine>;
```





##### pb_variable 或 Variable 和 Variable

为protoboard 和 ConstraintSystem上的约束系统上分配变量。通过在约束系统上分配变量，并且给变量赋予有限域的值，然后才能在约束系统上为这些变量构造约束。本质上一个Variable在约束系统上是通过一个index与其相互对应。变量分为private input以及public input。两者都是withness。

**libsnark**

分配private input 的方法是``

```c++
template<typename FieldT>
class variable {
public:

    var_index_t index;

    variable(const var_index_t index = 0) : index(index) {};
```

```c++
template<typename FieldT>
class pb_variable : public variable<FieldT> {
public:
    pb_variable(const var_index_t index = 0) : variable<FieldT>(index) {};

    void allocate(protoboard<FieldT> &pb, const std::string &annotation="");
};
```

**bellman**

```rust
/// Represents a variable in our constraint system.
#[derive(Copy, Clone, Debug)]
pub struct Variable(Index);
```





##### pb_variable_array 和 Vec\<Variable\>

约束系统上变量的数组。`pb_variable_array`类是继承于`private std::vector<pb_variable<FieldT> >`，这样做的好处是使得`pb_variable_array`使用起来更加方便。

```c++
template<typename FieldT>
class pb_variable_array : private std::vector<pb_variable<FieldT> >
{
    typedef std::vector<pb_variable<FieldT> > contents;
public:
    using typename contents::iterator;
    using typename contents::const_iterator;
    using typename contents::reverse_iterator;
    using typename contents::const_reverse_iterator;
    ......
```



##### linear_combination 和 Vec\<(Variable, E::Fr)\> 或者LinearCombination

表示$a*x_1 + b*x_2 + c*x_3$中的系数和变量的集合。linear_term 将相当于(x1, a)，(x2, b)，(x3, c)。linear_combination是3者的集合。pb_linear_conbination是对linear_combination 的一层封装，该类型可以是Variable，也可以是linear_combination类型。

```c++
/**
 * A linear term represents a formal expression of the form "coeff * x_{index}".
 */
template<typename FieldT>
class linear_term {
public:

    var_index_t index;
    FieldT coeff;
```

```c++
template<typename FieldT>
class pb_linear_combination : public linear_combination<FieldT> {
public:
// 如果为true，为variable，也就是(variable, coeff(Field::one())), 如果为false那么为std::vector<linear_term<FieldT> >的数组。
    bool is_variable;
    lc_index_t index;
```



**bellman**

```rust
/// This represents a linear combination of some variables, with coefficients
/// in the scalar field of a pairing-friendly elliptic curve group.
#[derive(Clone)]
pub struct LinearCombination<E: ScalarEngine>(Vec<(Variable, E::Fr)>);
```



##### pb_linear_combination_arrray 和 Vec<Vec\<(Variable, E::Fr)\>>

为一个线性组合的集合，相当于一个矩阵。

```c++
template<typename FieldT>
class pb_linear_combination_array : private std::vector<pb_linear_combination<FieldT> >
{
    typedef std::vector<pb_linear_combination<FieldT> > contents;
```







#### 参考链接

* https://zhuanlan.zhihu.com/p/78683739