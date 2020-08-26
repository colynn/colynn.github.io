---
title: HTTP 协议 - 理解
tags: ["http"]
categories: ["协议"]
date: 2020-08-24
lastmod: 2020-08-26
---


## 概述


Web 浏览器、服务器和相关的 Web应用程序都是通过 HTTP 相互通信的。HTTP是现代全球因特网中使用的公共语言。

HTTP使用的是可靠的数据传输协议，因此即使数据来自地球的另一端，它也能够确保数据在传输的过程中不会被损坏或产生混乱。

### 资源
* 统一资源标识符（Uniform Resource Identifier, URI）
* 统一资源定位符（URL)
* 统一资源名（URN)


### 协议版本

* HTTP/0.9  - 这个版本有严重设计权限

* http/1.0 - 广泛使用

* http/1.0+ 非官方的http/1.0的扩展版本

* http/1.1 目前正在使用的版本，修复的相关设计缺陷，增加的相关特性

* http-NG((又名 HTTP/2.0)) 将来使用与否正在商讨中

## 报文信息

HTTP报文是简单的格式化数据块，分为请求报文，响应报文。

* 请求报文

![image](https://user-images.githubusercontent.com/5203608/91008264-93195e80-e610-11ea-87b7-2bf0dac6f189.png)


* 响应报文

![image](https://user-images.githubusercontent.com/5203608/91008324-b6440e00-e610-11ea-90e1-116a1516ac61.png)


## 连接管理

连接管理是一个 HTTP 的关键话题：打开和保持连接在很大程度上影响着网站和 Web 应用程序的性能。在 HTTP/1.x 里有好些个模型：__短连接（short-lived connections）__, __持久连接（persistent connections）__, 和 __HTTP 管道（HTTP pipelining）__。


![image](https://user-images.githubusercontent.com/5203608/91302787-9449b780-e7d9-11ea-860a-f43ab399a02c.png)


### 持久连接

HTTP/1.1（以及 HTTP/1.0的各种增强版本）__允许 HTTP 设备在事务处理结束之后将 TCP 连接保持在打开状态__, 以便为未来的 HTTP 请求重用现存的连接。在事务处理结束之后仍然保持在打开状态的 TCP 连接被称为持久连接。非持久连接会在每个事务结束之后关闭。持久连接会在不同事务之间保持打开状态，直到客户端或服务决定将其关闭。

重用已对目标服务器打开的空闲持久连接，就可以避开缓慢的连接建立阶段。而且已经打开的连接还可以避免 __TCP慢启动__ 的拥塞适应阶段，以便更快速地进行数据的传输。