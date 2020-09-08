---
title: Go Mod 包管理
categories: ["Go"]
tags: ["go"]
date: 2019-08-15
lastmod: 2020-09-08
---

## 前言
In Go 1.11, a new tool has arrived. It's called go mod. For those who don't know what is that, I'll tell you about it.

One of the purposes of this tool is to get rid of ```$GOPATH```, so you can git clone a Golang repository anywhere and work on it without worrying about $GOPATH at all. There are other purposes of course, such as simpler versioning and usage, but I think the RIP ```$GOPATH``` is the most notable.

If the repository is a mod, all the tools you use regularly (```go get```, ```go build```, etc.) will handle it nicely without changing anything.


## 如何使用 Modules
把 golang 升级到 1.11（~~现在1.12 已经发布了，建议使用1.12~~）

设置 GO111MODULE

```go mod init [module-name]```. Yep, that's all you have to do. If you're trying to go mod-ify an existing project that uses $GOPATH, you should export the ```GO111MODULE=on``` variable (```go mod``` will tell you that as well).


## GO111MODULE

GO111MODULE 有三个值：off, on和auto（默认值）。

* GO111MODULE=off，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。
* GO111MODULE=on，go命令行会使用modules，而一点也不会去GOPATH目录下查找。
* GO111MODULE=auto，默认值，go命令行将会根据当前目录来决定是否启用module功能。这种情况下可以分为两种情形：
    * 当前目录在GOPATH/src之外且该目录包含go.mod文件
    * 当前文件在包含go.mod文件的目录下面。

__注__: 当modules 功能启用时，依赖包的存放位置变更为$GOPATH/pkg/mod，允许同一个package多个版本并存，且多个项目可以共享缓存的 module。


## How it work
再你初始化 go mod 后，如何让你的项目依赖一下第三方包，以大部分人都熟悉的beego为例吧！
修改Hello.go文件：

```go
package main

import "github.com/astaxie/beego"

func main() {
    beego.Run()
}
```

### 1.之前做法
要运行hello.go需要执行go get 命令 下载beego包到 ```$GOPATH/src```

### 2.当前推荐做法
直接 ``` go run hello.go```

稍等片刻… go 会自动查找代码中的包，下载依赖包，并且把具体的依赖关系和版本写入到go.mod和go.sum文件中。
查看go.mod，它会变成这样：

```go
module hello

go 1.12

require github.com/astaxie/beego v1.11.1
```

### 3. go.mod文件解释

require 关键子是引用，后面是包，最后v1.11.1 是引用的版本号； 这样，一个使用Go包管理方式创建项目的小例子就完成了。


## 问题一： 依赖包的版本是怎么控制的？
下载的依赖包 （```$GOPATH/pkg/mod```）是存在版本定义的（如：github.com/astaxie/beego@v1.11.1 ）最后会有一个版本号 1.11.1，也就是说，```$GOPATH/pkg/mod```里可以保存相同包的不同版本。

版本是在go.mod中指定的:
*  如果，在go.mod中没有指定，go命令会自动下载代码中的依赖的最新版本，本例就是自动下载最新的版本。

*  如果，在go.mod用require语句指定包和版本 ，go命令会根据指定的路径和版本下载包，
指定版本时可以用latest，这样它会自动下载指定包的最新版本；

### 依赖包的版本号是什么？ 

是包的发布者标记的版本号，格式为 vn.n.n (n代表数字)，本例中的beego的历史版本可以在其代码仓库release看到Releases · astaxie/beego · GitHub

如果包的作者还没有标记版本，默认为 v0.0.0


## 问题二： 依赖包中的地址失效了怎么办？

比如 golang.org/x/… 下的包都无法下载怎么办？
在go快速发展的过程中，有一些依赖包地址变更了。
以前的做法

修改源码，用新路径替换import的地址
git clone 或 go get 新包后，copy到$GOPATH/src里旧的路径下
无论什么方法，都不便于维护，特别是多人协同开发时。

使用go.mod就简单了，在go.mod文件里用 replace 替换包，例如

```go
replace golang.org/x/text => github.com/golang/text latest
```

这样，go会用 ```github.com/golang/text``` 替代```golang.org/x/text```，原理就是下载```github.com/golang/text``` 的最新版本到 ```$GOPATH/pkg/mod/golang.org/x/text```下。

## 问题三： init生成的go.mod的模块名称有什么用？
本例里，用 go mod init hello 生成的go.mod文件里的第一行会申明
module hello

因为我们的项目已经不在$GOPATH/src里了，那么引用自己怎么办？就用模块名+路径。

例如，在项目下新建目录 utils，创建一个tools.go文件:

```go
package utils

import “fmt”

func PrintText(text string) {
    fmt.Println(text)
}
```

在根目录下的hello.go文件就可以 import “hello/utils” 引用utils

```go
package main

import (
"hello/utils"

"github.com/astaxie/beego"
)

func main() {

    utils.PrintText("Hi")

    beego.Run()
}
```

## 问题四：以前老项目如何用新的包管理
1. 如果用```auto```模式，把项目移动到```$GOPATH/src```外
2. 进入目录，运行 ```go mod init [模块名称]```
3. ```go build``` 或者 ```go run``` 一次

## 问题五：mod形式下如何下载指定版本的包

```sh
// 注意：项目目录下操作
$ go get github.com/isbrick/tools@c87b277
$ go mod vendor
```
__注__: 根据官方的说法，从Go 1.13开始，模块管理模式将是Go语言开发的默认模式。
