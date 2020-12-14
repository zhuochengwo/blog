---
title: 从dep 迁移到Go Modules
tags: []
id: '470'
categories:
  - - Go
  - - 敲代码
date: 2019-04-09 19:02:58
---

### 动机

Go的依赖管理一直是被人诟病的，也有非常多的依赖管理工具，我之前一直用的官方的 `dep`，整体来说没有什么问题，只是项目需放在`GOPATH`下挺让人讨厌的，给人的感觉就是管的太多。 Go 在 `1.11` 中加入`Go Modules`，将会是 `Go 1.13` 中的默认行为，这也将是未来的Go依赖管理工具，我们也应该积极跟进，拥抱未来。

### 开启

> 可以用环境变量 GO111MODULE 开启或关闭模块支持，它有三个可选值：off、on、auto，默认值是 auto。 GO111MODULE=off 无模块支持，go 会从 GOPATH 和 vendor 文件夹寻找包。 GO111MODULE=on 模块支持，go 会忽略 GOPATH 和 vendor 文件夹，只根据 go.mod 下载依赖。 GO111MODULE=auto 在 $GOPATH/src 外面且根目录有 go.mod 文件时，开启模块支持。

在使用模块的时候，`GOPATH` 是无意义的，不过它还是会把下载的依赖储存在 `$GOPATH/pkg/mod` 中，也会把 `go install` 的结果放在 `$GOPATH/bin` 中。 **我们也可以不用管这个环境变量，默认的`auto`已经可以满足我们的需求了**

### 开始

我们先进入之前的项目，假设是`~/go/src/my_app` 在项目目录执行 `go mod init` 会自动处理依赖

```bash
cd ~/go/src/my_app
go mod init my_app #my_app是项目目录
```

可以看到如下输出

```vim
go: creating new go.mod: module my_app
go: copying requirements from Gopkg.lock
```

执行完后我们会看到创建了一个`go.mod`的文件，这里边就是我们项目的所有依赖信息

```vim
module my_app

go 1.11

require (
   github.com/gin-gonic/gin v1.3.0
   github.com/go-ini/ini v1.42.0
)
```

然后执行

```bash
go mod download
```

下载所有的依赖内容，这些依赖都会被下载到`$GOPATH/pkg/mod`中。 接下来你可以删除 `Gopkg.lock` 和 `Gopkg.toml` 文件，还有`vendor`目录，然后提交 `go.mod` 和 `go.sum` 文件。 关于`go mod`更多的使用方式可以使用

```bash
go help mod
```

### 注意

如果之前的项目结构是类似这种结构 . . └── my\_app ├── aaa │   ├── bbb │   ├── ccc │   │   ├── code.go │   ├── main.go 那么之前导入包可能就是这样

```go
import "my_app/aaa/ccc/code.go"
```

使用`Go Modules`后，最外层的目录就可以不需要了，因为不再是用`GOPATH`了。 所以导入要去掉`my_app`

```go
import "aaa/ccc/code.go"
```

### 最后

还是早点切换到`Go Modules`吧，毕竟`1.13`可就是默认方式了，`dep`等到时估计都要被淘汰了，Go的依赖管理也该统一了。