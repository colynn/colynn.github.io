---
title: HTTP 协议 - 理解
tags: ["http"]
categories: ["协议"]
date: 2020-08-24
lastmod: 2020-08-28
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



## 持久连接 (persistent connections)

HTTP/1.1（以及 HTTP/1.0的各种增强版本）__允许 HTTP 设备在事务处理结束之后将 TCP 连接保持在打开状态__, 以便为未来的 HTTP 请求重用现存的连接。在事务处理结束之后仍然保持在打开状态的 TCP 连接被称为持久连接。非持久连接会在每个事务结束之后关闭。持久连接会在不同事务之间保持打开状态，直到客户端或服务决定将其关闭。

重用已对目标服务器打开的空闲持久连接，就可以避开缓慢的连接建立阶段。而且已经打开的连接还可以避免 __TCP慢启动__ 的拥塞适应阶段，以便更快速地进行数据的传输。

### Keep-Alive

### 哑代理（blind relay）


## 管道化连接（HTTP Pipelining）