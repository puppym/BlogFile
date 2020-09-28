## rust 

* https://github.com/rust-in-blockchain/awesome-blockchain-rust 
* https://github.com/rust-cc/awesome-cryptography-rust

## rust or C++

C++ 

1. 要写接近实时，就不要能有垃圾回收(GC)。要想达到接近实时且高性能就需要接近底层，即所谓bare-metal。
2. 高性能可以用C达到，但考虑足够的开发效率，C的语言特性缺乏就是一个明显问题。
3. 从开发效率和可读可维护性上来说，足够的抽象能力是必须的，但这种抽象必须是没有运行时开销的(runtime cost)。
4. 从开发效率和可读可维护性上来说，足够的抽象能力是必须的，但这种抽象必须是没有运行时开销的(runtime cost)。

总体来说就是一个高性能的静态强类型多范式语言。但是C++的向前兼容也给它带来了一些历史的包袱。

rust 相比C++的新特性。

1. 无data-race的并发。
2. 函数式语言的pattern matching。
3. Generics和Trait粗看起来是zero cost abstraction的编译时多态。
4. 

## rust 语言特性

Rust 是一种 **预编译静态类型**（*ahead-of-time compiled*）语言，这意味着你可以编译程序，并将可执行文件送给其他人，他们甚至不需要安装 Rust 就可以运行。

```rust
// windows子系统安装
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

vim ~/.cargo/config

[registry]
index = "https://mirrors.ustc.edu.cn/crates.io-index/"
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index/"

设置中科大源，然后可以使用rustup来通过依赖源安装依赖包

```



## cargo 

```rust
cargo build  //构建项目
cargo run // 运行项目
cargo test // 测试项目
cargo doc //构建项目文档
cargo publish //可以将库发布到crates.io
cargo --version
```

