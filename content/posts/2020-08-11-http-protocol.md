---
title: HTTP 协议 - 理解
tags: ["http"]
categories: ["协议"]
date: 2020-08-24
lastmod: 2020-08-31
---


# 概述

Web 浏览器、服务器和相关的 Web应用程序都是通过 HTTP 相互通信的。HTTP是现代全球因特网中使用的公共语言。

HTTP使用的是可靠的数据传输协议，因此即使数据来自地球的另一端，它也能够确保数据在传输的过程中不会被损坏或产生混乱。

## 资源
* 统一资源标识符（Uniform Resource Identifier, URI）
* 统一资源定位符（URL)
* 统一资源名（URN)

![image](https://user-images.githubusercontent.com/5203608/91537527-0260bc80-e949-11ea-8eb8-118d3ba83ef2.png)

URL | URI |
------- | ----------------|
URL stands for Uniform Resource Locator.	| URI stands for Uniform Resource Identifier. |
URL is a subset of URI that specifies where a resource is exists and the mechanism for retrieving it.	 | A URI is a superset of URL that identifies a resource either by URL or URN (Uniform Resource Name) or both.
The main aim is to get the location or address of a resource	| The main aim of URI is to find a resource and differentiate it from other resources using either name or location.
URL is used to locate only web pages	| Used in HTML, XML and other files XSLT (Extensible Stylesheet Language Transformations) and more.
The scheme must be a protocol like HTTP, FTP, HTTPS, etc.	| In URI, the scheme may be anything like a protocol, specification, name, etc.
Protocol information is given in the URL.	| There is no protocol information given in URI.
Example of URL: https://google.com	| Example of URI: urn:isbn:0-486-27557-4
It contains components such as protocol, domain, path, hash, query string, etc.	| It contains components like scheme, authority, path, query, fragment component, etc. | 
All URLs can be URIs |	Not all URIs are URLs since a URI can be a name instead of a locator.

## 协议版本

* HTTP/0.9  - 这个版本有严重设计权限

* http/1.0 - 广泛使用

* http/1.0+ 非官方的http/1.0的扩展版本

* http/1.1 目前正在使用的版本，修复的相关设计缺陷，增加的相关特性

* http-NG((又名 HTTP/2.0)) 将来使用与否正在商讨中

# 报文信息

HTTP报文是简单的格式化数据块，分为请求报文，响应报文。

* 请求报文

![image](https://user-images.githubusercontent.com/5203608/91008264-93195e80-e610-11ea-87b7-2bf0dac6f189.png)


* 响应报文

![image](https://user-images.githubusercontent.com/5203608/91008324-b6440e00-e610-11ea-90e1-116a1516ac61.png)


# 连接管理

连接管理是一个 HTTP 的关键话题：打开和保持连接在很大程度上影响着网站和 Web 应用程序的性能。在 HTTP/1.x 里有好些个模型：__短连接（short-lived connections）__, __持久连接（persistent connections）__, 和 __HTTP 管道（HTTP pipelining）__。


![image](https://user-images.githubusercontent.com/5203608/91302787-9449b780-e7d9-11ea-860a-f43ab399a02c.png)


## 短连接 （short-lived connections)

HTTP 最早期的模型，也是 HTTP/1.0 的默认模型，是短连接。每一个 HTTP 请求都由它自己独立的连接完成；这意味着发起每一个 HTTP 请求之前都会有一次 TCP 握手，而且是连续不断的。

TCP 协议握手本身就是耗费时间的，所以 TCP 可以保持更多的热连接来适应负载。短连接破坏了 TCP 具备的能力，新的冷连接降低了其性能。

HTTP 事务的性能在很大程度上取决 于底层 TCP 通道的性能, 最常见的 TCP 相关时延主要包括如下：
* TCP 连接建立握手；
* TCP 慢启动拥塞控制；
* 数据聚集的 Nagle 算法；
* 用于捎带确认的 TCP 延迟确认算法；
* TIME_WAIT 时延和端口耗尽。

__注意__: 这是 HTTP/1.0 的默认模型(如果没有指定 Connection 协议头，或者是值被设置为 close)。而在 HTTP/1.1 中，只有当 Connection 被设置为 close 时才会用到这个模型。


## Connection 首部
HTTP 允许在客户端和最终的源端服务器之前存在一串 HTTP 中间实体（代理、高速缓存等）。可以从客户端开始，逐跳地将 HTTP 报文经过这些中间设备，转发到服务器上去（或者进行反向传输）。

Connection 首部可以承载3种不同类型的标签，因此有时会令人费解：
* HTTP 首部字段名，列出了只与此连接有关的首部；
* 任意标签值，用于描述此连接的非标准选项；
* 值 close, 说明操作完成之后需关闭这条持久连接。

__注意__: HTTP 的连接管理适用于两个连续节点之间的连接，是[`hop-by-hop`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers#hbh), 而不是[`end-to-end`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers#e2e)。 HTTP 协议头受不同连接模型的影响，比如 __Connection__ 和 __Keep-Alive__，就是 [`hop-by-hop`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers#hbh) 协议头，它们的值是可以被中间节点修改的。


## 长连接 ( keep-alive/ persistent connections)

HTTP/1.1（以及 HTTP/1.0的各种增强版本）__允许 HTTP 设备在事务处理结束之后将 TCP 连接保持在打开状态__, 以便为未来的 HTTP 请求重用现存的连接。在事务处理结束之后仍然保持在打开状态的 TCP 连接被称为持久连接。非持久连接会在每个事务结束之后关闭。持久连接会在不同事务之间保持打开状态，直到客户端或服务决定将其关闭。

重用已对目标服务器打开的空闲持久连接，就可以避开缓慢的连接建立阶段。而且已经打开的连接还可以避免 __TCP慢启动__ 的拥塞适应阶段，以便更快速地进行数据的传输。

### Keep-Alive

* http/1.0 keep-alive.  一种被称为 keep-alive连接的早期实验型持久连接， 存在一些互操作性设计方便问题的困扰。

__使用 Keep-alive 连接时有一些限制和一些需求澄清的地方。__
* http/1.0 keep-alive 并不是默认使用的。客户端必须发送一个  Connection: Keep-Alive 请求首部来激活 keep-alive 连接。
* 代理各网关必须执行 Connection 首部的规则。代理或网关必须在将报文转发出去或将其高速缓存之前，删除在 Connection 首部中命名的所有首部字段以及 Connection 首部自身。

* 从技术上来讲， 应该忽略所有来自 http/1.0设备的 Connection 首部字段（包括 Connection:Keep-Alive）, 因为它们可能是由比较老的代理服务器误转发的。但实际上，尽管可能会有在老代理上挂起的危险，有些客户端和服务呃呃还是会违反这条规则。

* 除百重复发送请求会产生其他一些副使用，否则如果 在客户端收到完整的响应之前连接就关闭了，客户端就一定要做好重试请求的准备。

### 盲中继（blind relay）

Web 客户端的 Connection: Keep-Alive 首部应该只会对这条离开客户端的 TCP 链路产生影响。这就是将其称作"连接"首部的原因, 那些不理解 Connection 首部，而且不知道在沿着转发链路将其发送出去之前，应该将该首部删除的 __代理__ .很多老的或是简单的代理都是盲中继（blind relay）, 它们只是将字节从一个连接转么至另一个连接中去，不对 Connection 首部进行特殊的处理。

![image](https://user-images.githubusercontent.com/5203608/91688505-b30ecc00-eb94-11ea-886c-0f1ef7a1befb.png)

为避免此类代理通信问题的发生，现代的代理都绝不能转发 Connection 首部和所有名字出现在 Connection 值中的首部。因此，如果一个代理收到了一个 Connection:Keep-Alive 首部，是不应该转发 Connection 首部，或所有名为 Keep-Alive 的首部的。

另外，还有几个不能作为 Connection 首部值列出，也不能被代理转发或作为缓存响应使用的首部。其中包括 Proxy-Authenticate、Proxy-Connection、Transfer-Encoding 和 Upgrade。

Proxy-Connection 的新首部，解决了在客户端后面紧跟着一个盲中断所带的问题，但并没有解决所有其他情况下存在的问题。

### HTTP/1.1持久连接 （persistent connections）
* http/1.1 逐渐停止了对 keep-alive 的支持，用一种名为持久连接（persistent connection）的改进型设计取代了它。持久连接的目的与 keep-alive 连接的目的相同，但工作机制更优一些。


http/1.1 默认就是持久连接的，协议头都不用再去声明它（但我们还是会把它加上，万一某个时候因为某种原因要退回到 HTTP/1.0 呢）。如果要在事务处理结束之后将连接关闭， HTTP/1.1应用程序必须向报文中显式地添加一个 Connection: close 首部。 但是，客户端和服务器仍然可以随时关闭空闲的连接。不发送 Connection: close __并不意味着服务承诺永远将连接保持在打开状态。__ 例如 Keep-Alive: timeout=5, max=100，表示这个TCP通道可以保持5秒，max=100，表示这个长连接最多接收100次请求就断开

持久连接与并行连接配合使用可能是最高效的方式。现在，很多 Web 应用程序都会打开少量的并行连接，其中的每一个都是持久连接。

Persistent HTTP connections 的优点, 下面是[RFC2616](http://tools.ietf.org/html/rfc2616)上总结:

- By opening and closing fewer TCP connections, CPU time is saved in routers and hosts (clients, servers, proxies, gateways, tunnels, or caches), and memory used for TCP protocol control blocks can be saved in hosts.

- HTTP requests and responses can be pipelined on a connection. Pipelining allows a client to make multiple requests without waiting for each response, allowing a single TCP connection to be used much more efficiently, with much lower elapsed time.

- Network congestion is reduced by reducing the number of packets caused by TCP opens, and by allowing TCP sufficient time to determine the congestion state of the network.

- Latency on subsequent requests is reduced since there is no time spent in TCP's connection opening handshake.

- HTTP can evolve more gracefully, since errors can be reported without the penalty of closing the TCP connection. Clients using future versions of HTTP might optimistically try a new feature, but if communicating with an older server, retry with old semantics after an error is reported.


__注意__:
1. HTTP 是一个无状态协议，这意味着每个请求都是独立的，Keep-Alive 没能改变这个结果。另外，Keep-Alive也不能保证客户端和服务器之间的连接一定是活跃的，在 HTTP1.1 版本中也如此。唯一能保证的就是当连接被关闭时你能得到一个通知，所以不应该让程序依赖于 Keep-Alive 的保持连接特性，否则会有意想不到的后果。
2. 使用长连接之后，客户端、服务端怎么知道本次传输结束呢？两部分：
    1. 判断传输数据是否达到了Content-Length 指示的大小；

    2. 动态生成的文件没有 Content-Length ，它是分块传输（chunked），这时候就要根据 chunked 编码来判断，chunked 编码的数据在最后有一个空 chunked 块，表明本次传输数据结束, 详见 - 附录1 Transfer-Encodin


__同keep-alive一样，持久连接的使用中也些限制和需要澄清的问题。__
* 发送了 Connection:close 请求首部之后，客户端就无法在那条连接上发送更多的请求了。
* 只有当连接上所有的报文都有正确的、自定义报文长度时————也就是说，实体主体部分的长度都和相应的 Connect-Length 一致，或者是用分块传输编码方式编码的————连接才能持久保持。
* 每个持久连接都只适用于一跳传输。
* 除非重复发起请求会产生副作用，否则如果在客户端收到整条响应之前连接关闭了，客户端就必须要重新发起请求。
* 一个用户客户端对任何服务或代理最多只能维护丙条持久连接，以防服务器过载。

## 管道化连接（HTTP Pipelining）

默认情况下，HTTP 请求是按顺序发出的。下一个请求只有在当前请求收到应答过后才会被发出。由于会受到网络延迟和带宽的限制，在下一个请求被发送到服务器之前，可能需要等待很长时间。

__流水线__ 是在同一条长连接上发出连续的请求，而不用等待应答返回。这样可以避免连接延迟。理论上讲，性能还会因为两个 HTTP 请求有可能被打包到一个 TCP 消息包中而得到提升。就算 HTTP 请求不断的继续，尺寸会增加，但设置 TCP 的 最大分段大小 MSS (Maximum Segment Size) 选项，任然足够包含一系列简单的请求。

并不是所有类型的 HTTP 请求都能用到流水线：只有 idempotent 方式，比如 GET、HEAD、PUT 和 DELETE 能够被安全的重试：如果有故障发生时，流水线的内容要能被轻易的重试。

今天，所有遵循 HTTP/1.1 的代理和服务器都应该支持流水线，虽然实际情况中还是有很多限制：一个很重要的原因是，任然没有现代浏览器去默认支持这个功能。


__HTTP 流水线在现代浏览器中并不是默认被启用的__：

* Web 开发者并不能轻易的遇见和判断那些搞怪的[代理服务器](https://en.wikipedia.org/wiki/Proxy_server)的各种莫名其妙的行为。
* 正确的实现流水线是复杂的：传输中的资源大小，多少有效的 [RTT](https://en.wikipedia.org/wiki/Round-trip_delay_time) 会被用到，还有有效带宽，流水线带来的改善有多大的影响范围。不知道这些的话，重要的消息可能被延迟到不重要的消息后面。这个重要性的概念甚至会演变为影响到页面布局！因此 HTTP 流水线在大多数情况下带来的改善并不明显。
* 流水线受制于 [HOL](https://en.wikipedia.org/wiki/Head-of-line_blocking) 问题。

由于这些原因，流水线已经被更好的算法给代替，如 multiplexing，已经用在 HTTP/2。

# HTTP/2的长连接与多路复用（multiplexing）

## 附录1 Transfer-Encoding

Transfer-Encoding 是一个用来标示 HTTP 报文传输格式的头部值。尽管这个取值理论上可以有很多，但是当前的 HTTP 规范里实际上只定义了一种传输取值——chunked。

如果一个HTTP消息（请求消息或应答消息）的Transfer-Encoding消息头的值为chunked，那么，消息体由数量未定的块组成，并以最后一个大小为0的块为结束。

每一个非空的块都以该块包含数据的字节数（字节数以十六进制表示）开始，跟随一个CRLF （回车及换行），然后是数据本身，最后块CRLF结束。在一些实现中，块大小和CRLF之间填充有白空格（0x20）。

最后一块是单行，由块大小（0），一些可选的填充白空格，以及CRLF。最后一块不再包含任何数据，但是可以发送可选的尾部，包括消息头字段。消息最后以CRLF结尾。

一个示例响应如下：
```
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

25
This is the data in the first chunk

1A
and this is the second one
0
```
注意：

* chunked 和 multipart 两个名词在意义上有类似的地方，不过在 HTTP 协议当中这两个概念则不是一个类别的。multipart 是一种 Content-Type，标示 HTTP 报文内容的类型，而 chunked 是一种传输格式，标示报头将以何种方式进行传输。
* chunked 传输不能事先知道内容的长度，只能靠最后的空 chunk 块来判断，因此对于下载请求来说，是没有办法实现进度的。在浏览器和下载工具中，偶尔我们也会看到有些文件是看不到下载进度的，即采用 chunked 方式进行下载。
* chunked 的优势在于，服务器端可以边生成内容边发送，无需事先生成全部的内容。HTTP/2 不支持 Transfer-Encoding: chunked，因为 HTTP/2 有自己的 streaming 传输方式（Source：[MDN - Transfer-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding)）。



## 参考
1. [HTTP 权威指南](https://www.ituring.com.cn/book/844)
2. [HTTP/1.x 的连接管理](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Connection_management_in_HTTP_1.x)
3. [HTTP Headers](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)
