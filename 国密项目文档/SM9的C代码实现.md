# SM9的C代码实现

代码中的过程和文档中的参数命名有差异，导致不好理解。2. 代码中的一些接口比较原始，不好和文档对应。

## 代码结构

```bash
SM9_FREE
├── Makefile
├── SM9Test.c
├── SM9文档 //SM9的文档，写的比官方文档要好懂一点
├── miracl //大数运算的基础库，为椭圆曲线有限域的运算提供基础
└── sm9
    ├── print_out.c // 格式化打印
    ├── print_out.h
    ├── sm3.c       //SM3杂凑哈希
    ├── sm3.h       
    ├── sm4.c       //SM4对称加解密
    ├── sm4.h
    ├── sm9_algorithm.h  
    ├── sm9_encrypt.c  // SM9加解密
    ├── sm9_keyexchange.c //SM9密钥交换
    ├── sm9_setup.c
    ├── sm9_signature.c    //SM9签名和验签
    ├── sm9_utils.c        //SM9的工具函数，比如H1和H2的杂凑函数
    ├── sm9_utils.h
    ├── sm_r-ate.c         // R-ate双线性对的运算
    ├── smzzn12.c          // 为R-ate双线性对提供底层的双线性对计算
    └── smzzn12.h
```



## 密钥生成

```C++
SM9_GenMSignPubKey() -> 
```



## 签名流程

```c++
SM9_Sign_New() -> SM9_Signature() -> zzn12_pow(_MIPP_ &w, r)(w = g^r) ->  Hfun()(h = H2(M||w, N)) 杂凑函数 -> subtract()(r-h) -> divide(_MIPP_ r, sm9n, sm9n) (l = (r-h)modN) 
```



## 验签流程



## 加解密过程



## 密钥交换





## Miracl库

MIRACL(Multiprecision Integer and Rational Arithmetic C/c++ Library)是一套由Shamus Software Ltd.所开发的一套关于大数运算函数库，用来设计与大数运算相关的密码学之应用，包含了RSA 公开密码学、Diffie-Hellman密钥交换(Key Exchange)、AES、DSA数字签名，还包含了较新的椭圆曲线密码学(Elliptic Curve Cryptography)等等。运算速度快，并提供源代码。国外著名密码学函数库还有：GMP、NTL、Crypto++、LibTomCrypt(LibTomMath)、OpenSSL等。



## 参考文档

* https://blog.csdn.net/yaoyuanyylyy/article/details/80871509
* 