## derive

手动实现编译器默认已经实现的行为。

The following is a list of derivable traits:

- Comparison traits: [`Eq`](https://doc.rust-lang.org/std/cmp/trait.Eq.html), [`PartialEq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html), [`Ord`](https://doc.rust-lang.org/std/cmp/trait.Ord.html), [`PartialOrd`](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html).
- [`Clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html), to create `T` from `&T` via a copy.
- [`Copy`](https://doc.rust-lang.org/core/marker/trait.Copy.html), to give a type 'copy semantics' instead of 'move semantics'.
- [`Hash`](https://doc.rust-lang.org/std/hash/trait.Hash.html), to compute a hash from `&T`.
- [`Default`](https://doc.rust-lang.org/std/default/trait.Default.html), to create an empty instance of a data type.
- [`Debug`](https://doc.rust-lang.org/std/fmt/trait.Debug.html), to format a value using the `{:?}` formatter.

https://doc.rust-lang.org/rust-by-example/trait/derive.html#derive



## rust--问号操作符

```rust
// 什么是问号操作符?
// 参考: https://doc.rust-lang.org/book/second-edition/ch09-02-recoverable-errors-with-result.html
// 参考: https://stackoverflow.com/questions/42917566/what-is-this-question-mark-operator-about


// 由于Rust中没有Exception异常处理的语法,
// Rust只有panic报错, 并且panic不允许被保护, 因为没有提供 try 这种语法.
// Rust的异常处理是通过 Result 的 Ok 和 Err 成员来传递和包裹错误信息.
// 然而错误信息的处理一般都是要通过match来对类型进行比较, 所以很多时候
// 代码比较冗余, 通过?符号来简化Ok和Err的判断.


// 下面的例子提供了一个不使用?符号 以及 一个使用?符号的样例代码.
fn halves_if_even<'a >(i: i32) -> Result<i32, &'a str> {                       // 取数值的二分之一.
    if i % 2 == 0 {
        Ok(i/2)
    } else {
        Err("error")
    }
}

fn not_use_question_mark() {
    let a = 10;                                                                // 把这里改成 9 就会报错.
    let half = halves_if_even(a);
    let half = match half {
        Ok(item) => item,
        Err(e) => panic!(e),
    };
    assert_eq!(half, 5);
}


fn use_question_mark<'a >() -> Result<i32, &'a str> {                          // 这里必须要返回Result
    let a = 10;
    let half = halves_if_even(a)?;                                             // 因为?要求其所在的函数必须要返回Result
    assert_eq!(half, 5);
    Ok(half)                                                                   
}


fn main() {
    not_use_question_mark();
    let _ = use_question_mark();
}
```

## cfg

可以通过cfg来实现条件编译，具体的写法有以下两种操作：

- the `cfg` attribute: `#[cfg(...)]` in attribute position
- the `cfg!` macro: `cfg!(...)` in boolean expressions

```rust
// This function only gets compiled if the target OS is linux
#[cfg(target_os = "linux")]
fn are_you_on_linux() {
    println!("You are running linux!");
}

// And this function only gets compiled if the target OS is *not* linux
#[cfg(not(target_os = "linux"))]
fn are_you_on_linux() {
    println!("You are *not* running linux!");
}

fn main() {
    are_you_on_linux();
    
    println!("Are you sure?");
    if cfg!(target_os = "linux") {
        println!("Yes. It's definitely linux!");
    } else {
        println!("Yes. It's definitely *not* linux!");
    }
}
```

## extern crate

连接`crate`到该新库，该种写法不仅仅连接该库，而且导入与库同名的模块下的所有项。模块可见性规则也用于库。

```rust
// Link to `library`, import items under the `rary` module
extern crate rary;

fn main() {
    rary::public_function();

    // Error! `private_function` is private
    //rary::private_function();

    rary::indirect_access();
}
```

##  Option

Option有时用于代替`panic!`捕获错误，这种写法能够通过`Option`枚举实现。

The `Option<T>` enum has two variants:

- `None`, to indicate failure or lack of value, and
- `Some(value)`, a tuple struct that wraps a `value` with type `T`.

```rust
// An integer division that doesn't `panic!`
fn checked_division(dividend: i32, divisor: i32) -> Option<i32> {
    if divisor == 0 {
        // Failure is represented as the `None` variant
        None
    } else {
        // Result is wrapped in a `Some` variant
        Some(dividend / divisor)
    }
}

// This function handles a division that may not succeed
fn try_division(dividend: i32, divisor: i32) {
    // `Option` values can be pattern matched, just like other enums
    match checked_division(dividend, divisor) {
        None => println!("{} / {} failed!", dividend, divisor),
        Some(quotient) => {
            println!("{} / {} = {}", dividend, divisor, quotient)
        },
    }
}

fn main() {
    try_division(4, 2);
    try_division(1, 0);

    // Binding `None` to a variable needs to be type annotated
    let none: Option<i32> = None;
    let _equivalent_none = None::<i32>;

    let optional_float = Some(0f32);

    // Unwrapping a `Some` variant will extract the value wrapped.
    println!("{:?} unwraps to {:?}", optional_float, optional_float.unwrap());

    // Unwrapping a `None` variant will `panic!`
    println!("{:?} unwraps to {:?}", none, none.unwrap());
}
```

## | |闭包

在rust中闭包称为匿名表达式`lambda`能够捕获封闭的环境，例如下面的闭包能够捕获x变量

`|val| val + x`

闭包相当于匿名函数，其调用方式也和一个函数一样，输入和输出类型能够被编译系统自动推测，当时输入变量的名字必须明确，便于在闭包中使用：

闭包的其它特点包括：

* 用`||`代替`()`包括输入变量。
* **可选的**函数体用`{}`包括。
* 拥有捕获外部环境变量的能力。

```rust
fn main() {
    // Increment via closures and functions.
    fn  function            (i: i32) -> i32 { i + 1 }

    // Closures are anonymous, here we are binding them to references
    // Annotation is identical to function annotation but is optional
    // as are the `{}` wrapping the body. These nameless functions
    // are assigned to appropriately named variables.
    let closure_annotated = |i: i32| -> i32 { i + 1 };
    let closure_inferred  = |i     |          i + 1  ;

    let i = 1;
    // Call the function and closures.
    println!("function: {}", function(i));
    println!("closure_annotated: {}", closure_annotated(i));
    println!("closure_inferred: {}", closure_inferred(i));

    // A closure taking no arguments which returns an `i32`.
    // The return type is inferred.
    let one = || 1;
    println!("closure returning one: {}", one());

}
```

## trait

相当于C++或者go里面的interface，一个父类可以由多个子类来实现。trait为未知类型收集一些方法。通过`self.`可以访问在同一个`trait`的其它方法。

对任何数据类型都可以实现 trait。在下面例子中，我们定义了包含一系列方法 的 `Animal`。然后针对 `Sheep` 数据类型实现 `Animal` `trait`，因而 `Sheep` 的实例可以使用 `Animal` 中的所有方法。

```rust
struct Sheep { naked: bool, name: &'static str }

trait Animal {
    // 静态方法签名；`Self` 表示实现者类型（implementor type）。
    fn new(name: &'static str) -> Self;

    // 实例方法签名；这些方法将返回一个字符串。
    fn name(&self) -> &'static str;
    fn noise(&self) -> &'static str;

    // trait 可以提供默认的方法定义。
    fn talk(&self) {
        println!("{} says {}", self.name(), self.noise());
    }
}

impl Sheep {
    fn is_naked(&self) -> bool {
        self.naked
    }

    fn shear(&mut self) {
        if self.is_naked() {
            // 实现者可以使用它的 trait 方法。
            println!("{} is already naked...", self.name());
        } else {
            println!("{} gets a haircut!", self.name);

            self.naked = true;
        }
    }
}

// 对 `Sheep` 实现 `Animal` trait。
impl Animal for Sheep {
    // `Self` 是实现者类型：`Sheep`。
    fn new(name: &'static str) -> Sheep {
        Sheep { name: name, naked: false }
    }

    fn name(&self) -> &'static str {
        self.name
    }

    fn noise(&self) -> &'static str {
        if self.is_naked() {
            "baaaaah?"
        } else {
            "baaaaah!"
        }
    }
    
    // 默认 trait 方法可以重载。
    fn talk(&self) {
        // 例如我们可以增加一些安静的沉思。
        println!("{} pauses briefly... {}", self.name, self.noise());
    }
}

fn main() {
    // 这种情况需要类型标注。
    let mut dolly: Sheep = Animal::new("Dolly");
    // 试一试 ^ 移除类型标注。

    dolly.talk();
    dolly.shear();
    dolly.talk();
}
```

## 泛型

定义泛型类型或泛型函数之类的东西时，我们会用 `<A>` 或者 `<T>` 这类标记 作为类型的代号，就像函数的形参一样。在使用时，为把 `<A>`、`<T>` 具体化，我们 会把类型说明像实参一样使用，像是 `<i32>` 这样。这两种把（泛型的或具体的）类型 当作参数的用法就是**类型参数**。

例如定义一个名为 `foo` 的 **泛型函数**，它可接受类型为 `T` 的任何参数 `arg`：

```rust
fn foo<T>(arg: T) { ... }
```

因为我们使用了泛型类型参数 `<T>`，所以这里的 `(arg: T)` 中的 `T` 就是泛型 类型。即使 `T` 在之前被定义为 `struct`，这里的 `T` 仍然代表泛型。

```rust
// 一个具体类型 `A`。
struct A;

// 在定义类型 `Single` 时，第一次使用类型 `A` 之前没有写 `<A>`。
// 因此，`Single` 是个具体类型，`A` 取上面的定义。
struct Single(A);
//            ^ 这里是 `Single` 对类型 `A` 的第一次使用。

// 此处 `<T>` 在第一次使用 `T` 前出现，所以 `SingleGen` 是一个泛型类型。
// 因为 `T` 是泛型的，所以它可以是任何类型，包括在上面定义的具体类型 `A`。
struct SingleGen<T>(T);

fn main() {
    // `Single` 是具体类型，并且显式地使用类型 `A`。
    let _s = Single(A);
    
    // 创建一个 `SingleGen<char>` 类型的变量 `_char`，并令其值为 `SingleGen('a')`
    // 这里的 `SingleGen` 的类型参数是显式指定的。
    let _char: SingleGen<char> = SingleGen('a');

    // `SingleGen` 的类型参数也可以隐式地指定。
    let _t    = SingleGen(A); // 使用在上面定义的 `A`。
    let _i32  = SingleGen(6); // 使用 `i32` 类型。
    let _char = SingleGen('a'); // 使用 `char`。
}

```

## 属性

属性是应用于某些模块、crate 或项的元数据（metadata）。这元数据可以用来：

- [条件编译代码](https://rustwiki.org/zh-CN/rust-by-example/attribute/cfg.html)
- [设置 crate 名称、版本和类型（二进制文件或库）](https://rustwiki.org/zh-CN/rust-by-example/attribute/crate.html)
- 禁用 [lint](https://en.wikipedia.org/wiki/Lint_(software)) （警告）
- 启用编译器的特性（宏、全局导入（glob import）等）
- 链接到一个非 Rust 语言的库
- 标记函数作为单元测试
- 标记函数作为基准测试的某个部分

当属性作用于整个 crate 时，它们的语法为 `#![crate_attribute]`，当它们用于模块 或项时，语法为 `#[item_attribute]`（注意少了感叹号 `!`）。

属性可以接受参数，有不同的语法形式：

- `#[attribute = "value"]`
- `#[attribute(key = "value")]`
- `#[attribute(value)]`

属性可以多个值，它们可以分开到多行中：

```rust
#[attribute(value, value2)]

#[attribute(value, value2, value3,
            value4, value5)]
```

## where

可以使用`where`分句来表达约束，它放在 `{` 的前面，而不需写在类型第一次出现之前。另外 `where` 从句可以用于任意类型的限定，而不局限于类型参数本身。

`where`的使用场景主要有以下：

* 当分别指定泛型的类型和约束时会更加清楚：

```rust
impl <A: TraitB + TraitC, D: TraitE + TraitF> MyTrait<A, D> for YourType {}

// 使用 `where` 从句来表达约束
impl <A, D> MyTrait<A, D> for YourType where
    A: TraitB + TraitC,
    D: TraitE + TraitF {}
```



* 当使用 `where` 从句比正常语法更有表现力时。本例中的 `impl` 如果不用 `where` 从句，就无法直接表达。

```rust
use std::fmt::Debug;

trait PrintInOption {
    fn print_in_option(self);
}

// 这里需要一个 `where` 从句，否则就要表达成 `T: Debug`（这样意思就变了），
// 或着改用另一种间接的方法。
impl<T> PrintInOption for T where
    Option<T>: Debug {
    // 我们要将 `Option<T>: Debug` 作为约束，因为那是要打印的内容。
    // 否则我们会给出错误的约束。
    fn print_in_option(self) {
        println!("{:?}", Some(self));
    }
}

fn main() {
    let vec = vec![1, 2, 3];

    vec.print_in_option();
}
```

## &self 和 self 的区别

&self，表示向函数传递的是一个引用，不会发生对象所有权的转移；
self，表示向函数传递的是一个对象，会发生所有权的转移，对象的所有权会传递到函数中。

```rust
#[derive(Debug)]
struct MyType {
    name: String
}

impl MyType {
    fn do_something(self, age: u32) {
    //等价于 fn do_something(self: Self, age: u32) {
    //等价于 fn do_something(self: MyType, age: u32) {
        println!("name = {}", self.name);
        println!("age = {}", age);
    }

    fn do_something2(&self, age: u32) {
        println!("name = {}", self.name);
        println!("age = {}", age);
    }
}

fn main() {
    let my_type = MyType{name: "linghuyichong".to_string()};
    //使用self
    my_type.do_something(18);   //等价于MyType::do_something(my_type, 18);
    //println!("my_type: {:#?}", my_type);    //在do_something中，传入的是对象，而不是引用，因此my_type的所有权就转移到函数中了，因此不能再使用

    //使用&self
    let my_type2 = MyType{name: "linghuyichong".to_string()};
    my_type2.do_something2(18);
    my_type2.do_something2(18);
    println!("my_type2: {:#?}", my_type2);//在do_something中，传入是引用，函数并没有获取my_type2的所有权，因此此处可以使用
    println!("Hello, world!");
}

```

## Self和self的区别

所有的trait都定义了一个隐式的类型Self，它指当前实现此接口的类型。相当于是实现trait的类型。

**self**

当`self`用作函数的第一个参数时，它等价于`self: Self`。`&self`参数等价于`self: &Self`。`&mut self`参数等价于`self: &mut Self`。

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height  //结构实例，"."
    }
}
```



**Self**

方法参数中的`Self`是一种语法糖，是方法的接收类型（例如，本方法所在的`impl`的类型）。

它可能出现在`trait`或`impl`中。但经常出现在`trait`中，它是任何最终实现`trait`的类型代替（在定义`trait`时未知该类型）。

```rust
example1:
trait Clone {
    fn clone(&self) -> Self;
}

impl Clone for MyType {
    //我可以使用具体类型(当前已知的，例如MyType)
    fn clone(&self) -> MyType;

    //或者再次使用Self，毕竟Self更简短
    fn clone(&self) -> Self
}

example2:
trait T {
    type Item;  //关联类型
    const C: i32;  //常量，书里没讲，更没在特性里见过。说明特性可以有常量

    fn new() -> Self;  //实施者的类型

    fn f(&self) -> Self::Item;  //实施时决定
}

struct S; //空结构

impl T for S {
    type Item = i32;  //具体化
    const C: i32 = 9;  //特性里的数据赋值

    fn new() -> Self {           // 类型S
        S
    }

    fn f(&self) -> Self::Item {  // i32
        Self::C                  // 9, 此时没有实例，实施的是类型
    }
}

```

**super**

```rust
mod a {
    pub fn foo() {}
}
mod b {
    pub fn foo() {
        super::a::foo(); // 父模块
    }
}
fn main() {}

//串联
mod a {
    fn foo() {}

    mod b {
        mod c {
            fn foo() {
                super::super::foo(); // call a's foo function
                self::super::super::foo(); // call a's foo function
            }
        }
    }
}
fn main() {}
```

## std::result::Result

```rust
[+] Expand attributes
pub enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`Result` is a type that represents either success ([`Ok`](https://doc.rust-lang.org/std/result/enum.Result.html#variant.Ok)) or failure ([`Err`](https://doc.rust-lang.org/std/result/enum.Result.html#variant.Err)).

See the [`std::result`](https://doc.rust-lang.org/std/result/index.html) module documentation for details.

## 返回Option中Some包含值

`value.unwrap()`返回Option中Some的包含值。



## rust代码运算符重载

在 Rust 中，很多运算符可以通过 trait 来重载。也就是说，这些运算符可以根据它们的 输入参数来完成不同的任务。这之所以可行，是因为运算符就是方法调用的语法糖。例 如，`a + b` 中的 `+` 运算符会调用 `add` 方法（也就是 `a.add(b)`）。这个 `add` 方 法是 `Add` trait 的一部分。因此，`+` 运算符可以被任何 `Add` trait 的实现者使用。

会重载运算符的 `trait`（比如 `Add` 这种）可以在[这里](http://doc.rust-lang.org/core/ops/)查看。

```rust
use std::ops;

struct Foo;
struct Bar;

#[derive(Debug)]
struct FooBar;

#[derive(Debug)]
struct BarFoo;

// `std::ops::Add` trait 用来指明 `+` 的功能，这里我们实现 `Add<Bar>`，它是用于
// 把对象和 `Bar` 类型的右操作数（RHS）加起来的 `trait`。
// 下面的代码块实现了 `Foo + Bar = FooBar` 这样的运算。
impl ops::Add<Bar> for Foo {
    type Output = FooBar;

    fn add(self, _rhs: Bar) -> FooBar {
        println!("> Foo.add(Bar) was called");

        FooBar
    }
}

// 通过颠倒类型，我们实现了不服从交换律的加法。
// 这里我们实现 `Add<Foo>`，它是用于把对象和 `Foo` 类型的右操作数加起来的 trait。
// 下面的代码块实现了 `Bar + Foo = BarFoo` 这样的运算。
impl ops::Add<Foo> for Bar {
    type Output = BarFoo;

    fn add(self, _rhs: Foo) -> BarFoo {
        println!("> Bar.add(Foo) was called");

        BarFoo
    }
}

fn main() {
    println!("Foo + Bar = {:?}", Foo + Bar);
    println!("Bar + Foo = {:?}", Bar + Foo);
}
```

## Fn, FnMut, FnOnce

虽然 Rust 无需类型说明就能在大多数时候完成变量捕获，但在编写函数时，这种模糊写法 是不允许的。当以闭包作为输入参数时，必须指出闭包的完整类型，它是通过使用以下 `trait` 中的一种来指定的。其受限制程度按以下顺序递减：

- `Fn`：表示捕获方式为通过引用（`&T`）的闭包
- `FnMut`：表示捕获方式为通过可变引用（`&mut T`）的闭包
- `FnOnce`：表示捕获方式为通过值（`T`）的闭包

> 译注：顺序之所以是这样，是因为 `&T` 只是获取了不可变的引用，`&mut T` 则可以改变 变量，`T` 则是拿到了变量的所有权而非借用。

## collect()，from()， into()

collect()主要用于错误收集， from()允许一个类型定义如何从另一个类型创建自身，用于原始类型和公共类型的转换。into()如果某一类型实现了From，那么可以直接获得该实现。

```rust
use std::convert::From;

#[derive(Debug)]
struct Number {
    value: i32,
}

impl From<i32> for Number {
    fn from(item: i32) -> Self {
        Number { value: item }
    }
}
// into()
fn main() {
    let int = 5;
    // Try removing the type declaration
    let num: Number = int.into();
    println!("My number is {:?}", num);
}
//from()
fn main() {
    let num = Number::from(30);
    println!("My number is {:?}", num);
}
```

## async/await

async 和 await 是可以分开的两个术语，分开理解（针对 Rust 语言的，其他语言就很不一样）

1. async：产生一个 Future 对象，一个没有任何作用的对象，必须由调用器调用才会有用

2. await: 等待异步操作完成（基于语义理解，其实很多情况只有调用 future.await 才是事实上去调用，具体是不是之前就开始执行，这个要看我们的调用器是什么），这步是阻塞当前线程，这个语法属于 Future 对象才能调用，而且必须要在 async 函数内。

   这列出三个异步基础作用

1. 延迟计算：不需要立即得到结果
2. 并发：多个操作并发执行，像多线程执行操作
3. 独立性：

