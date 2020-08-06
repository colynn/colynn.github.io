---
title: 搭建npm私有镜像仓库
gh-repo: Kubernetes-Best-Pratice/verdaccio
gh-badge: [star, fork, follow]
categories: ["教程"]
tags: ["npm", "tools"]
date: 2019-11-29
lastmod: 2019-11-29
---

## 前言
当你的研发团队越来越大，或是你无法忍受node超慢的构建时你可以考虑继续读下去，给大家推荐一个基于[Verdaccio](https://verdaccio.org/docs/en/what-is-verdaccio)相对较完整的解决方案。

由于环境的原因，我们直接去 `npmjs.org` 下载就不要考虑了，可以将`npm config set registry=https://registry.npm.taobao.org` 可以缓解一部分, 但是如果你有些自己公司定制的npm包如何在公司内分享呢，这个时候你就需要一个npm私服了. 


## Verdaccio 是什么
Verdaccio 是一个简单的零配置的node.js轻量私有的npm代理仓库。

## verdaccion能做什么
### 1. 缓存npmjs.org仓库

npm安装缓慢大家都知道，可以使用它来减少延迟（大概“慢”的npmjs.org每个软件包/版本仅连接一次）并提供有限的故障转移（如果使用npmjs.org挂掉了，我们依然可以通过缓存获取到相应的包）

### 2. 私有仓库

如果您想在公司中使用npm软件包系统而又不将所有代码发送给公众，请使用私有软件包，就像使用公共软件包一样容易。

### 3. 链接多个仓库

如果您在组织中使用多个npm仓库，并且需要在一个项目中从多个来源获取软件包，则可以利用Verdaccio的上行链路功能，将多个仓库链接起来并从一个端点获取。

### 4. 覆盖公共软件包

如果要使用某个第三方软件包的修改版本（例如，您发现了一个错误，但维护者尚未接受请求请求），则可以使用相同的名称在本地发布该版本。 详细请看[这里](https://verdaccio.org/docs/en/best#override-public-packages)。

### 5. 端到端测试
事实证明，Verdaccio是一个轻量级仓库，可以在几秒钟内启动，对于任何CI来说都足够快。 许多开放源代码项目都使用verdaccio进行端到端测试.

_注_: 功能3、4、5需要你自己去探索。

## 安装

_注_: Verdaccio 支持多种安装方式，甚至包含charts包

### 前置条件
#### 1. Node版本

* verdaccio@3.x Node v6.12 是最低的版本.
* verdaccio@4.0.0-alpha.x or verdaccio@4.x Node 8.x (LTS "Carbon") 是最低的版本.

#### 2. npm >=4.x or yarn

官方强烈建议使用最新的node包管理客户端 > npm@5.x | yarn@1.x | pnpm@2.x

#### 3. Web界面 支持 Chrome, Firefox, Edge, and IE11 browsers.


### Docker-compose方式部署
可以直接参考我们归档的版本[传送门](https://github.com/Kubernetes-Best-Pratice/verdaccio)

![image](https://user-images.githubusercontent.com/5203608/89489083-3d555180-d7dc-11ea-9647-98de3419354e.png)


是时候舍弃cnpm了，希望你可以享受verdaccio. 


## 更多
* 请参考[verdaccio官方文档](https://verdaccio.org/docs/en/configuration)

* 提速node的构建速度，建议启用[多阶段构建](https://www.freecodecamp.org/news/speed-up-node-re-builds-leveraging-docker-multi-stage-builds-and-save-money-65189a4ab115/)

## 参考链接
1. https://medium.com/engenharia-noalvo/ways-to-have-your-private-npm-registry-and-a-final-diy-solution-eed001a88e74

2. https://verdaccio.org/docs/en/what-is-verdaccio