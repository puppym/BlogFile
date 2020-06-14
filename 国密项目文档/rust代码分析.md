# Rust代码分析

本文主要分析cita的SM2实现，然后通过SM2的实现理清楚SM9的实现思路：

## SM2代码分析

### 代码结构

```rust
ecc.rs   // 椭圆曲线上群的相关运算
field.rs // 椭圆曲线上域的相关运算
mod.rs   // 向外部暴露的接口
signature.rs // SM2具体的签名流程
```

### ecc.rs

主要的结构体为`EccCtx`

* new() 新建一个`EccCtx`对象
* inv_n(&self, x: &BigUint)
* new_point(&self, x: &FieldElem, y: &FieldElem) 新建一个Point
* new_jacobian(&self,x: &FieldElem,y: &FieldElem,z: &FieldElem,) 新建一个jacobian点
* generator(&self) 根据指定参数生成一个Point
* to_affine(&self, p: &Point) 
* neg(&self, p: &Point) Point点取负数
* add(&self, p1: &Point, p2: &Point) 两个点相加
* 

`lazy_static!`:`lazy_static`提供了一个宏`lazy_static!`，使用这个宏把你的静态变量“包裹”起来就可以实现延迟初始化了，实际上这个宏会帮助你生成一个特定的`struct`,这个`struct`的`deref`方法(trait Deref)提供了延迟初始化的能力，它也提供了`initialize`方法，你也可以在代码中主动地调用它进行初始化。

### field.rs

主要结构体为`FieldCtx`主要函数如下：

```rust
pub struct FieldCtx {
    modulus: FieldElem,
    modulus_complete: FieldElem,
}
```

下面的有限域上的运算都是调用的`raw_add`等接口，下面的接口只是暴露在最外层使用。

* new()：新建一个fieldCtx
* add(&self, a: &FieldElem, b: &FieldElem) ：FieldElem 元素的加法
* sub(&self, a: &FieldElem, b: &FieldElem)：fieldElem元素减法
* fast_reduction(&self, input: &[u32; 16])：fieldElem元素快速减法
* mul(&self, a: &FieldElem, b: &FieldElem)：fieldElem元素乘法
* square(&self, a: &FieldElem)：fieldElem元素平方
* cubic(&self, a: &FieldElem)：fieldElem元素3次方
* inv(&self, x: &FieldElem)：x^-1
* neg(&self, x: &FieldElem)：有限域元素取反
* exp(&self, x: &FieldElem, n: &BigUint)：
* sqrt(&self, g: &FieldElem)：有限域元素开方

最里层有限域的运算：

```rust
pub struct FieldElem {
    pub value: [u32; 8],
}
```

* raw_add(a: &FieldElem, b: &FieldElem) 
* raw_sub(a: &FieldElem, b: &FieldElem)
* u32_mul(a: u32, b: u32) (inline)
* raw_mul(a: &FieldElem, b: &FieldElem)
* new(x: [u32; 8])
* from_slice(x: &[u32]) vec[] -> FieldElem
* zero()
* ge(&self, x: &FieldElem)  self >= x
* eq(&self, x: &FieldElem)
* div2(&self, carry: u32)
* is_even(&self)
* to_bytes(&self)

...

### mod.rs



### signature.rs

```rust
pub struct Signature {
    r: BigUint,
    s: BigUint,
}
```

`Signature`的成员函数如下：

* der_decode(buf: &[u8])  解码
* der_decode_raw(buf: &[u8])  原始解码
* der_encode(&self)



```rust
pub struct SigCtx {
    curve: EccCtx,
}
```

`SigCtx`的成员函数如下：

* new()
* hash(&self, id: &str, pk: &Point, msg: &[u8]) 获取杂凑值e
* sign(&self, msg: &[u8], sk: &BigUint, pk: &Point)  对外部暴露的签名函数
* sign_raw(&self, digest: &[u8], sk: &BigUint) 内部的签名函数
* verify(&self, msg: &[u8], pk: &Point, sig: &Signature) 对外部暴露的签名验证函数
* verify_raw(&self, digest: &[u8], pk: &Point, sig: &Signature) 内部的签名验证函数
* new_keypair(&self) 新建密钥对
* pk_from_sk(&self, sk: &BigUint) 从sk求出pk
* load_pubkey(&self, buf: &[u8]) byte类型的数组转换为Point点，并且返回
* serialize_pubkey(&self, p: &Point, compress: bool) 序列化Point点的pulicKey
* load_seckey(&self, buf: &[u8])  byte转化为bigint，加载私钥
* serialize_seckey(&self, x: &BigUint)  序列化私钥BigUint转化为Vec<u8>

### 数据转换过程

![rust代码](C:\Users\czm18\Desktop\备份\blog\国密项目文档\rust代码.png)

### new_keypair()代码流程

```bash
curve.random_uint()（产生随机数sk（私钥）） -> curve->g_mul()（pk = sk*G 得出公钥pk）->校验pk是否符合条件 -> 返回密钥对
```

### sign()流程

这里需要特别强调一下，当前`rust`代码中所有的`HASH_256`哈希都是指代的是SM3哈希函数。

```bash
ctx.sign() -> self.hash() -> hasher.get_hash() (Z_A = HASH_256(ID_LEN || ID || x_G || y_G || x_A || y_A)) ->  hasher.get_hash() (e = HASH_256(Z_A || M)) -> self.sign_raw(&digest[..], sk)计算签名过程
```

### verify()流程

```bash
ctx.verify() -> self.hash() -> hasher.get_hash() (Z_A = HASH_256(ID_LEN || ID || x_G || y_G || x_A || y_A)) ->  hasher.get_hash() (e = HASH_256(Z_A || M)) -> self.verify_raw()
```





