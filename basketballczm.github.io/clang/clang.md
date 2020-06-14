# clang的安装和使用
* https://clang.llvm.org/get_started.html

# clang常用命令
1. 产生LLVM IR文件：clang -Os -S -emit-llvm hello.cpp -o hello.ll
2. 进一步将LLVM IR编译成汇编文件

# clang 前端的工作步骤
1. 词法分析。
2. AST类的代码分析。
3. 解析程序，将不同的单词转换为AST树上的节点。(语法分析就是将各个符号进行组合之后会形成表达式，声明和函数体。最后形成AST树)
4. 顶层的解析源代码的工作。
5. 主函数驱动代码。