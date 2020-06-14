## rust的安装

rust的安装参考以下的几个链接:

* https://www.rust-lang.org/zh-CN/tools/install
* https://networm.me/2018/07/01/vscode-rust/
* https://www.jianshu.com/p/af6a59d644eb

注意在vscode中安装rust的相关插件时候应该注意rust编译器的版本与vscode中配置文件的版本相互匹配。stable，nightly，beta。

当前的配置文件如下：

```
    "rust-client.rustupPath":"/home/czm/.cargo/bin/rustup",
    "rust.rustsymPath":"/home/czm/.cargo/bin/rustsym",
    "rust-client.channel": "stable",
    "rust.mode": "legacy",
    "rust.rustup": {
        "toolchain": "stable-x86_64-unknown-linux-gnu",
        "stableToolchain": "stable-x86_64-unknown-linux-gnu"
    },
    "rust.rustfmtPath": "/home/czm/.cargo/bin/rustfmt",
    "[rust]": {
        "editor.defaultFormatter": "rust-lang.rust"
    },
    "rust.rls": {
        "useRustfmt": true
    },
```

