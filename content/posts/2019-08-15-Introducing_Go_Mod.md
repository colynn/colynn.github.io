---
title: Go Mod 包管理 & gRPC、Protobuf扫盲
categories: []
tags: ["go", "go mod", "grpc", "protoc"]
date: 2019-08-15
lastmod: 2025-02-25
---

## 前言

In Go 1.11, a new tool has arrived. It's called go mod. For those who don't know what is that, I'll tell you about it.

One of the purposes of this tool is to get rid of ```$GOPATH```, so you can git clone a Golang repository anywhere and work on it without worrying about $GOPATH at all. There are other purposes of course, such as simpler versioning and usage, but I think the RIP ```$GOPATH``` is the most notable.

If the repository is a mod, all the tools you use regularly (```go get```, ```go build```, etc.) will handle it nicely without changing anything.

## 如何使用 Modules

把 golang 升级到 1.11

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

直接 ```go run hello.go```

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

* 如果，在go.mod中没有指定，go命令会自动下载代码中的依赖的最新版本，本例就是自动下载最新的版本。

* 如果，在go.mod用require语句指定包和版本 ，go命令会根据指定的路径和版本下载包，
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

## 问题六：Go 如何 import private的代码仓库的包

对于 public 的仓库，大家知道是可以直接import的，而对于 private 代码仓库我们则需要如下操作：

### 对于本地开发环境

1. The Because of go module proxy site just like Maven default repo go also has a proxy site(<https://proxy.golang.org,direct>), 所以我们需要通过声明`GOPRIVATE`环境变量来绕过, 如果 `GOPRIVATE`有多个值通过逗号来分隔。

```sh
go env -w GOPRIVATE=git.repoxxx.com/[groupName]
```

__注__: __The new `GOPRIVATE` environment variable indicates module paths that are not publicly available.__ It serves as the default value for the lower-level GONOPROXY and GONOSUMDB variables, which provide finer-grained control over which modules are fetched via proxy and verified using the checksum database.

2. 添加 access_token或是 ssh key 解决私有仓库的验证问题。

```sh
# access_token 可以在对应用户下配置
$ git config --global url."https://${username}:${access_token}@private.gitrepo.com".insteadOf /
"https://private.gitrepo.com"

# Or use ssh-key 将私钥放置下对应的用户下
# 也许，这个 ssh-key 私钥的并不是默认路径，那么你可以通过这个方式指定
# $ cat ~/.ssh/config
# Host yourserver
#     Hostname something.domain.tld
#     IdentityFile /var/www/html/ma.ttias.be/.ssh/id_rsa
#     IdentitiesOnly yes
$ git config --global url."git@yourserver".insteadOf /
"https://yourserver"
```

__特别说明__： 不论使用`access_token`或是`ssh key`, 强烈建议使用独立的用户，另外对于步骤2中insteadOf 的地址建议尽量再细, 避免其他的同站下的git项目产生影响, 如下：

```sh
$ cat ~/.gitconfig  |grep url -A3
[url "http://goget:b9e6b3cafbf5789d74bdce16@gogs.domain.com/rudder/drone.git/dist"]
 insteadOf = http://gogs.domain.com/rudder/drone.git/dist
```

### 对于CI/CD

如果你的项目已经将`vendor`随代码一起提交，那么你 go build 时可以直接用 `-mod vendor`的方式来 build, 倘若你的项目里没有管理`vendor`项目，那么`Dockerfile`里也要有类似于 __对于本地开发环境__ 的设置。

`Dockerfile`部分示例

```Dockerfile
...
RUN go env -w GOPRIVATE=github.com/colynn
# 确认 build 环境里包含 git command
RUN apk add git
RUN git config --global url."https://golang:<access-token>@github.com/colynn/tools".insteadOf "https://github.com/colynn/tools"
...
```

## 问题七：go mod 如何不确定依赖是从哪里引入的，可以使用

```sh
go mod why -m <依赖名>
```

## gRPC 扫盲

* __最重要的注意各组件版本间的兼容性__

### Protobuf（Protocol Buffers）

Protobuf 是 Google 开发的一种数据序列化协议，用于高效地传输结构化数据。与 JSON 或 XML 等格式相比，Protobuf 具有更高的效率和更小的二进制大小。

主要特点：

* 平台和语言无关：Protobuf 支持多种编程语言（如 Go、Java、C++ 等）。
* 高效：相比于 JSON 或 XML，Protobuf 的序列化和反序列化速度更快，生成的数据更小。
* 强类型：Protobuf 定义了一个非常强类型的语言，能精确描述消息结构（字段类型、字段顺序、默认值等）。

### Protoc

* `Protoc`（Protocol Buffers Compiler）是一个用于将 `.proto` 文件（定义服务和消息格式的文件）编译成特定语言代码的工具，是Protobuf的编译器；
* 它是所有 Protobuf 项目的基础，用于生成编解码（serialization/deserialization）代码。

### protoc-gen-go

* `protoc-gen-go` 是 Protobuf 用于生成 Go 语言代码的 __插件__。
* 它从 `.proto` 文件生成 Go 语言文件，通常这些文件包含了用于序列化、反序列化、以及实现 gRPC 服务所需要的 Go 代码。
* `protoc-gen-go` 是 Protobuf 编译器的扩展插件，必须与 Protoc 一起使用。
* `protoc-gen-go-grpc`（Go 语言的 gRPC 代码生成插件）：用于生成 gRPC 服务器和客户端代码（从 protoc-gen-go 中拆分出来）。
* `go install google.golang.org/protobuf/cmd/protoc-gen-go@latest`
* `go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest`

### protoc-gen-validate

* `github.com/envoyproxy/protoc-gen-validate`
* 一个 Protocol Buffers（Protobuf）校验插件，用于在 Protobuf `.proto` 文件中定义字段级别的 数据验证规则，并生成相应的 Go（或其他语言）的代码，以在运行时进行数据校验。
* `github.com/envoyproxy/protoc-gen-validate` 曾经是 PGV（Protobuf 校验插件）的官方仓库，但现在已经被迁移到了 `github.com/bufbuild/protoc-gen-validate`。

### gRPC
  
* gRPC 是由 Google 开发的一个高性能、开源和通用的远程过程调用（RPC）框架。gRPC 通过 Protobuf 作为接口定义语言，支持跨语言的服务调用, 基于 HTTP/2 协议并使用 Protobuf 作为序列化协议。
* gRPC 使用 protoc (Protobuf的编译器)生成的代码进行服务定义，并提供接口来执行客户端和服务端之间的通信。
* 跨语言支持：gRPC 支持多种语言，像是 Go、Java、Python、C++ 等。
* 双向流：gRPC 支持客户端和服务器之间的双向流通信，适合实时通信和大规模系统。

### gRPC-go

* The Go language implementation of gRPC. HTTP/2 based RPC
* （即 gRPC 的 Go 语言实现）是 gRPC 官方提供的 Go 语言版本实现，允许开发者在 Go 环境中使用 gRPC。它与 gRPC 的核心功能相同，但专门针对 Go 语言进行了封装和优化。

### grpc-gateway

* grpc-gateway 是一个用于生成 RESTful HTTP API 接口的工具，基于 gRPC 服务。
* 它允许你使用 HTTP/JSON 通过代理调用 gRPC 服务，主要用于为 gRPC 服务提供兼容的 RESTful API 接口（比如从 Web 客户端访问 gRPC 服务）。
* grpc-gateway v2: v2兼容 gRPC v1.40+ 及更高版本，v2 改进了 runtime.HTTPError 处理方式

### protoc-gen-grpc-gateway

* __作为 protoc 插件__，自动生成 HTTP API 转发到 gRPC 服务器的代码。
* 让 gRPC 服务能通过 HTTP+JSON 访问，而不是只能使用 gRPC 客户端。

* 确保 `grpc-gateway` 和 `protoc-gen-grpc-gateway` 版本匹配
  * v2.x 需要 google.golang.org/protobuf
  * v1.x 需要 github.com/golang/protobuf
  * 不能单独使用 protoc-gen-grpc-gateway, 需要配合 protoc-gen-go-grpc，否则无法生成 gRPC 代码。
  * v2 latest版本: `go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@latest`

### OpenAPI(Swagger)

* v1: v1 使用 `protoc-gen-swagger` 生成的 OpenAPI 2.0 规范 JSON 需要手动调整
* `go install github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger@v1.16.0`
* v2: `protoc-gen-swagger` 在 gRPC-Gateway v2 被 `protoc-gen-openapi` 取代 (protoc使用示例 `--openapiv2_out .  --openapiv2_opt logtostderr=true \`), 默认支持 OpenAPI 3.0，并改进了 JSON Schema 结构
* `go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapi@latest`
  
### go.mod的pkg解释

* `google.golang.org/grpc`: 是 Go 语言的 gRPC 框架
  * 生成 gRPC 客户端和服务器 的核心库。
  * 提供 gRPC 连接管理（grpc.Dial）。
  * 支持 gRPC 的流式通信、拦截器、中间件等功能。
  * `protoc-gen-go-grpc` 依赖 grpc-go，要求__`grpc v1.64.0`__ 及以上。
  
* __`google.golang.org/protobuf`__: Protocol Buffers 的 Go 语言实现
  * 提供 proto.Message 接口，定义 Protobuf 消息的基本操作。
  * 支持 Protobuf 的序列化、反序列化（proto.Marshal、proto.Unmarshal）。
  * 处理 `.pb.go` 代码文件的核心库，所有 .proto 生成的 Go 代码都依赖它。
  * `protoc-gen-go` v1.20+ 之后，protoc-gen-go 开始使用 google.golang.org/protobuf 作为 Protobuf 运行时库，取代了老的 `github.com/golang/protobuf`。

| 组件    | 作用 |    版本要求   |
| -------- | ------- |------- |
| protoc | 编译 .proto 文件的核心工具 | 建议 v3.14+，支持新 protoc-gen-go 代码 |
| protoc-gen-go | 生成 .pb.go 文件的插件 | v1.27+ 需要 `google.golang.org/protobuf`, 兼容 grpc-gateway v2|
| protoc-gen-go-grpc | 生成 .pb.go 的 gRPC 代码 | v1.3.0+ 兼容 grpc |
| protoc-gen-grpc-gateway | 生成 REST API 网关代码, 依赖 `protoc-gen-go-grpc`，必须先生成 gRPC 代码 | 注意v1/v2版本对于protobuf的库依赖不同 |
| google.golang.org/protobuf | Protobuf 运行时库 | v1.34.0+ 兼容新 protoc-gen-go |
| google.golang.org/grpc | gRPC 服务器/客户端核心库 | v1.40+ 兼容 protoc-gen-go-grpc |
| github.com/grpc-ecosystem/grpc-gateway/v2| Generates a reverse-proxy server which translates a RESTful JSON API into gRPC. |  注意v1/v2对于protoc的库版本的依赖不同 |

### protoc命令示例

```sh
# grpc-gateway v1
protoc -I $GOPATH/pkg/mod/github.com/grpc-ecosystem/grpc-gateway@v1.16.0/third_party/googleapis \
       -I $GOPATH/pkg/mod/github.com/envoyproxy/protoc-gen-validate@v0.6.7/validate \
       -I $proto_package \
       --proto_path=$proto_package "$protofile" \
       --go_out=paths=source_relative:"$pb_go_package" \
       --go-grpc_out=paths=source_relative:$pb_go_package \
       --grpc-gateway_out=logtostderr=true:allow_delete_body=true,paths=source_relative:$pb_go_package \
       --swagger_out=logtostderr=true:$swagger_package
```

```sh
# grpc-gateway v2 
# you should git clone googleapis firstly
# git clone https://github.com/googleapis/googleapis.git $GOPATH/src/github.com/googleapis/googleapis
protoc  -I$GOPATH/src/github.com/googleapis/googleapis \
    -I$(go env GOPATH)/pkg/mod/github.com/envoyproxy/protoc-gen-validate@0.6.7/validate \
    -I $proto_package \
    --go_out=paths=source_relative:"$pb_go_package" \
    --go-grpc_out=paths=source_relative:$pb_go_package \
    --grpc-gateway_out=logtostderr=true,allow_delete_body=true,paths=source_relative:$pb_go_package \
    --grpc-gateway_opt=logtostderr=true,allow_delete_body=true \
    --openapiv2_out=logtostderr=true:$swagger_package
```

### [TODO] what it bufbuild
