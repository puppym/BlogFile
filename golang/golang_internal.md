报错：

```bash
main.go:26:2: use of internal package github.com/ethereum/go-ethereum/internal/debug not allowed
```

[internal](https://stackoverflow.com/questions/33351387/how-to-use-internal-packages)

一个go库函数包中的internal文件夹下面的包是不能够被导入的，详细见上面的文档。

```
An import of a path containing the element “internal” is disallowed if the importing code is outside the tree rooted at the parent of the “internal” directory.
```

对于第三方库中的internal不能被别的库引用。

---------------

更新：

报错：

```golang
czm@DESKTOP-2R5G5OF:~/go/src/github.com/ethereum/go-ethereum$ make geth
build/env.sh go run build/ci.go install ./cmd/geth
build/ci.go:61:2: use of internal package github.com/ethereum/go-ethereum/internal/build not allowed
Makefile:15: recipe for target 'geth' failed
make: *** [geth] Error 1
```

上面放的路径没有问题，但是报错说不能用internal包，但是很奇怪，该internal是本身go-ethereum就自带的，按理说应该能够使用。[internal报错](https://github.com/jpmorganchase/quorum/issues/909) 通过链接中`unset GOPROXY`和`unset GO111MODULE`之后能够正常编译，可能的原因是编译过程中直接默认通过代理去查找第三方包。默认引用的是第三方的internal包。

-------

关于go跳不能正常跳转，如果配置啥的都正确，尝试等一会在进行不能跳转函数的跳转，可能函数正在构建。

可以尝试更新gopls

```
GO111MODULE=on 
go get golang.org/x/tools/gopls@latest.
```

---

更新

go 跳转频繁奔溃的原因是设置了远程了调试，remote setting json.