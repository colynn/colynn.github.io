---
title: HTTP 协议 - 理解
tags: ["http"]
categories: ["协议"]
date: 2020-08-24
lastmod: 2020-08-24
---


## 基础知识

### 具体含义

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

### 持久连接
