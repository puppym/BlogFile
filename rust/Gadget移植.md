# bellman gadget移植

本文主要介绍在bellman中如何构建`minGadget`。

![comparsion_gadget](C:\Users\czm18\Desktop\备份\blog\rust\comparsion_gadget.png)

**rust 实现MinGadget思路**

结合当前bellman中已有的gadget，当前应该还要实现以下内容：

- LeqGadget
- TernaryGadget
- ComparsionGadget
- packing_gadget
- disjunction_gadget

下面将分别实现上面的gadget。对于comparsion_gadget的约束主要主要有n+6个约束，详细约束过程参考https://secbit.feishu.cn/file/boxcn1VZ6cp9k51EoxMBz5TNPKX文档。

## ternary_gadget

### 实现过程

一般gadget的实现过程如下：

1. 首先看一下AllocatedNum结构体的定义如下：（每一类gadget都有自己的结构体，该结构体代表每类gadget的输入）

   ```rust
   pub struct AllocatedNum<E: ScalarEngine> {
       value: Option<E::Fr>, //代表约束系统中一个变量的值
       variable: Variable,   //代表约束系统中的一个变量
   }
   ```

   value和variable。分别计算两者的值，variable的值需要`cs.alloc`函数进行分配（selected），value的值是在计算初始化variable的过程中计算的。

2. 根据电路中gadget输入和输出变量之间的关系，构造约束等式比如`selected = b?x:y`的约束等式为`b*(y-x)=y-selected`。该约束等式可以通过一定的代数关系推得，比如`boolean.rs`中的`xor`gadget。

3. 根据约束等式通过`cs.enforce`函数将约束等式添加到CS约束系统中。

4. 返回`AllocatedNum`对象。对于`TernaryGadget`而言其返回的AllocatedNum对象的value和variable的值已经在前面进行初始化，返回过程中只需用该两个变量构造一个`AllocatedNum`对象即可。

#### 约束

$b * (y-x) = y-selected$

对应的具体代码如下：

```rust
/*
pub struct AllocatedNum<E: ScalarEngine> {
    value: Option<E::Fr>, //代表约束系统中一个变量的值
    variable: Variable,   //代表约束系统中的一个变量
}
*/
pub fn TernaryGadget<CS>(
        &self,
    // 约束系统变量
        mut cs: CS,
    // 
        x: &Self,
        y: &Self,
    ) -> Result<Self, SynthesisError>
    where
        CS: ConstraintSystem<E>,
    {
        // 步骤1定义和初始化AllocatedNum中的value以及variable变量
        let mut value = None;
        let selected = cs.alloc(
            || "the result",
            || {
                let mut tmp = *self.value.get()?;
                value = if tmp == E::Fr::one() {
                    x.value.clone()
                } else {
                    y.value.clone()
                };
                Ok(value.unwrap())
            },
        )?;
        // 步骤2构造TernaryGadget的输入变量和输出变量之间的约束
        /*
        Constraint:
        select = b?x:y;
        b * (y-x) = y-selected;
        */
        
        // 步骤3将步骤2构造的输入和输出变量之间的约束通过cs.enforce加入CS约束系统中。
        cs.enforce(
            || "TernaryGadget constraint",
            |lc| lc + self.variable,
            |lc| lc + y.variable - x.variable,
            |lc| lc + y.variable - selected,
        );
        // 步骤4返回结果的AllocatedNum变量
        Ok(AllocatedNum {
            value,
            variable: selected,
        })
    }
```

**gadget test**

```rust
    #[test]
    fn test_ternary_gadget() {
        {
            // 第一个例子b值为1，因此结果值为5
            let mut cs = TestConstraintSystem::<Bls12>::new();
            // 在CS约束系统中的名字为“a”的namespace中分配一个值为1的变量
            let b_true = AllocatedNum::alloc(cs.namespace(|| "a"), || Ok(Fr::from_str("1").unwrap())).unwrap();
            let x = AllocatedNum::alloc(cs.namespace(|| "c"), || Ok(Fr::from_str("5").unwrap())).unwrap();
            let y = AllocatedNum::alloc(cs.namespace(|| "d"), || Ok(Fr::from_str("10").unwrap())).unwrap();
            let res_true = b_true.TernaryGadget(&mut cs, &x, &y).unwrap();
            // 断言cs约束系统的可满足性
            assert!(cs.is_satisfied());
            // 下面两行是两种方式获取结果变量的值
            // cs.get函数可以获取cs约束系统中名为“the result”变量()，来校验结果的正确性。
            assert!(cs.get("the result") == Fr::from_str("5").unwrap());
            // 返回的AllocatedNum中包含结果变量的值
            assert!(res_true.value.unwrap() == Fr::from_str("5").unwrap());
            // set一个错误的结果变量，查看约束系统是否依然成立。
            cs.set("the result", Fr::from_str("20").unwrap());
            assert!(!cs.is_satisfied());
        }
        {
            // 第二个例子b值为0，因此结果值为10
            let mut cs = TestConstraintSystem::<Bls12>::new();
            let b_false = AllocatedNum::alloc(cs.namespace(|| "b"), || Ok(Fr::from_str("0").unwrap())).unwrap();
            let x = AllocatedNum::alloc(cs.namespace(|| "c"), || Ok(Fr::from_str("5").unwrap())).unwrap();
            let y = AllocatedNum::alloc(cs.namespace(|| "d"), || Ok(Fr::from_str("10").unwrap())).unwrap();
    
            let res1_false = b_false.TernaryGadget(&mut cs, &x, &y).unwrap();
            assert!(cs.is_satisfied());
            assert!(cs.get("the result") == Fr::from_str("10").unwrap());
            assert!(res1_false.value.unwrap() == Fr::from_str("10").unwrap());
    
            cs.set("the result", Fr::from_str("20").unwrap());
            assert!(!cs.is_satisfied());
        }
    }
```



### 问题

**println!大法好**

1. 在写测试的过程中没有注意namespace的概念。比如：

错误的写法

```rust
            let b_true = AllocatedNum::alloc(&mut cs, || Ok(Fr::from_str("1").unwrap())).unwrap();
            let x = AllocatedNum::alloc(&mut cs, || Ok(Fr::from_str("5").unwrap())).unwrap();
            let y = AllocatedNum::alloc(&mut cs, || Ok(Fr::from_str("10").unwrap())).unwrap();
```

上面的主要问题在于在同一个`root_namespace`中`alloc`多个变量，会报错变量重定义，这里为什么一个namespace只能定义一个变量？具体原因还不太清楚。

正确的写法

```rust
            let b_true = AllocatedNum::alloc(cs.namespace(|| "a"), || Ok(Fr::from_str("1").unwrap())).unwrap();
            let x = AllocatedNum::alloc(cs.namespace(|| "c"), || Ok(Fr::from_str("5").unwrap())).unwrap();
            let y = AllocatedNum::alloc(cs.namespace(|| "d"), || Ok(Fr::from_str("10").unwrap())).unwrap();
```

2. 在同一个CS中同时测试b为true和false的情况？这样做是不行的，原因在于这样会在同一个CS中构建同一个变量selected "the result"，在运行时刻会崩溃。

## disjunction_gadget

### 实现过程

在bellman中实现该gadget是融合了libsnark中的disjunctionGadget和packingGadget的约束过程。

对于packingGadget的约束为

1. $1*\sum  bits[i] * 2^i = alpha\_packed$  **注意这里一共要计算i+1位二进制，第n+1位为less_or_equal的二进制表示形式**
2. 对每个bit位的约束，该位只可能为0或者1。$(1 - bits_i) * bits_i = 0$

对于disjunction的约束为

1. 对于二进制比特位的和和其相反数，两者乘积为output约束。$inv * sum = output$
2. 对于输出和二进制比特位和的约束。$(1-output)*sum = 0$



## comparsion_gadget

该gadget主要有3个约束，分别如下：

1. 对disjunction_gadget的返回值进行约束   $(1-output)*output=0$
2. 对比较计算的结果值进行约束  $ 1 * (2^n + B - A) = alpha\_packed$ 通过第n+1位来判断$less\_or\_equal$的值，通过第0到第n位的和来判断 $less$的值。
3.  添加约束$less\_or\_equal * output = less$ 三者的关系满足该约束。

```bash
      packed(alpha) = 2^n + B - A
      
      // not_all_zeros 为alpha的0到n-1位的二进制的和。
      not_all_zeros = \bigvee_{i=0}^{n-1} alpha_i

      if B - A > 0, then 2^n + B - A > 2^n,
          so alpha_n = 1 and not_all_zeros = 1
      if B - A = 0, then 2^n + B - A = 2^n,
          so alpha_n = 1 and not_all_zeros = 0
          
      if B - A < 0, then 2^n + B - A \in {0, 1, \ldots(......), 2^n-1},
          so alpha_n = 0

      therefore alpha_n = less_or_eq and alpha_n * not_all_zeros = less
```

**具体代码实现见num.rs**



## bellman和libsnark中的变量对比

##### libsnark变量说明

1. variable

   ```C++
    template <typename FieldT>
   
    class variable {
        public:
           var_index_t index;
           ...
    };
   ```

   varible 的定义非常简单，描述一个 variable，只需要记录一个 varible 对应的标号就行了。比如对应编号为 index 的 variable，表示的是 $x_{index}$变量。

2. linear_term

   linear_term 描述了一个线性组合中的一项。线性组合中的一项由变量以及对应的系数组成：

   ```C++
   template <typename FieldT>
   class linear_term {
    public:
        var_index_t index;
        FieldT coeff;
        ...
    };
   ```

3. linear_combination

   linear_combination 描述了一个完整的线性组合。一个 linear combination 由多个 linear term 组成：

   ```c++
    template <typename FieldT>
    class linear_combination {
        public:
           std::vector<linear_term<FieldT> > terms;
    ...
    };
   ```



##### bellman变量说明

1. variable

   ```rust
   pub struct LinearCombination<E: ScalarEngine>(Vec<(Variable, E::Fr)>); //linear_term项数组
   ```

2. linear_term

   ```rust
   (Variable, E::Fr) // 系数加变量(index索引)
   ```

3. linear_combination

   ```rust
   pub struct LinearCombination<E: ScalarEngine>(Vec<(Variable, E::Fr)>); // 下标index索引
   ```


##### 两者对比

|      | bellman                                                      | libsnark                                 |
| ---- | ------------------------------------------------------------ | ---------------------------------------- |
|      | pub struct LinearCombination<E: ScalarEngine>(Vec<(Variable, E::Fr)>); | pb_linear_combination(linear_term项数组) |
|      | (Variable, E::Fr)                                            | linear_term(系数加变量(index索引))       |
|      | pub struct Variable(Index);                                  | pb_variable(下标index索引)               |

​      

$S_i = \sum_{k=0}^{i+1} A[i] * B[i]$