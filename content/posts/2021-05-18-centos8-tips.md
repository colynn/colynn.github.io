---
title: Centos 8 Tips
tags: ["运维", "centos8"]
categories: []
date: 2021-05-18
lastmod: 2021-05-18
---


## 系统yum源调整

```sh
# centos 8
$ wget -O /etc/yum.repos.d/CentOS-Base.repo  http://mirrors.aliyun.com/repo/Centos-8.repo
```

```sh
# centos 7
$ wget -O /etc/yum.repos.d/CentOS-Base.repo  http://mirrors.aliyun.com/repo/Centos-7.repo
```

```sh
# centos 6
$ wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```

## Active ip addr on centos 8

```sh
nmcli c r eth0  ; nmcli d r eth0
```