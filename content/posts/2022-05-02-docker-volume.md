---
title: 理解并正确使用docker volume/bind mount
tags: ["docker"]
categories: []
date: 2022-05-04
lastmod: 2022-05-05
---


## 写在前面

默认情况下，在容器内创建的所有文件都存储在一个可写的容器层上(如下图)，

![Image](https://user-images.githubusercontent.com/5203608/166707397-5e8d89b4-78b9-4813-a67e-c180a083af98.png)

### 问题
但是这也带来如下的几个问题：
* 当该容器不再存在时，数据并不持久，而且如果另一个进程需要这些数据，就很难将其从容器中取出。
* 容器的可写层与容器所运行的主机紧密相连。你不能轻易地把数据移到别的地方。
* 写入容器的可写层需要一个存储驱动来管理文件系统。存储驱动提供了一个联合文件系统，使用Linux内核。与使用数据卷直接写在主机文件系统中，这种额外的抽象降低了性能。


### 解决方案
`Docker`对于容器文件的持久化提供了两种方式：`volumes`、`bind mounts`

![image](https://user-images.githubusercontent.com/5203608/166710239-5dbfaf24-67bc-4c76-b287-0c8c625141d0.png)

tips: 
* __Volumes__ are stored in a part of the host filesystem which is managed by Docker (`/var/lib/docker/volumes/` on Linux). Non-Docker processes should not modify this part of the filesystem. Volumes are the best way to persist data in Docker.

* __Bind mounts__ may be stored anywhere on the host system. They may even be important system files or directories. Non-Docker processes on the Docker host or a Docker container can modify them at any time.

* __tmpfs mounts__ are stored in the host system’s memory only, and are never written to the host system’s filesystem.


## Volumes

### 介绍

通过阅读docker的官方文档其实感觉到一直尽力地告诉你`volumes`是docker数据持久化的首选机制。

对于volume有两种使用方式， `named volumes`/ `anonymous volumes`. 匿名卷（`anonymous volumes`）在第一次挂载到容器中时没有明确的名称，所以Docker给它们一个随机的名称，保证在特定的Docker主机中是唯一的。 除了名字, 命名卷与匿名卷其他的表现均是一样的。

`Volumes`另外还支持使用`Volume drivers`，允许你将数据存储在远程主机或云供应商，以及其他可能性。

### volumes in real world

我们基于`mysql`的容器来做一个说明，我们创建一个 `data_volume`的volumes 用来存储mysql的数据， 如下图：

```sh
# create data_volume for mysql container
docker volume create data_volume
docker run -v data_volume:/var/lib/mysql mysql

# 如果不提前创建 data_volume2, docker会自动创建这个volume
docker run -v data_volume2:/var/lib/mysql mysql 
```

![image](https://user-images.githubusercontent.com/5203608/166905854-e0c845ad-407c-42bf-ac0a-d74e87abb35a.png)


## Bind mounts

对于数据持久化`bind mounts`在`Docker`的早期就已经存在了，挂载的文件或目录不需要在Docker主机上已经存在。如果它还不存在就会按需创建。绑定挂载的性能非常好，但它们依赖于主机的文件系统有一个特定的目录结构。

但是如果你正在开发新的Docker应用程序，请考虑使用`named volumes`.

### bind mounts in real world

```sh
docker run -v /data/mysql:/var/lib/mysql mysql
```
![image](https://user-images.githubusercontent.com/5203608/166928022-f67cb581-9142-4f9f-b30e-1c22dc5d6457.png)

### tips
* __Bind mounts allow access to sensitive files__
One side effect of using bind mounts, for better or for worse, is that you can change the host filesystem via processes running in a container, including creating, modifying, or deleting important system files or directories. This is a powerful ability which can have security implications, including impacting non-Docker processes on the host system.


## 关于挂载方式 -v or --mount

### 概述
`-v` 是旧的一种命令方式，建议使用`--mount`选项，因为它更详细，必须以key/value的格式来指定每个参数（其实这个你也可以理解为缺点，有些麻烦）。

```sh
# volumes
docker run \
--mount type=volume, source=data_volume, target=/var/lib/mysql mysql
```

```sh
# bind mounts
docker run \
--mount type=bind, source=/data/mysql, target=/var/lib/mysql mysql
```

### 参数说明

* The `type` of the mount, which can be `bind`, `volume`, or `tmpfs`. 
* The `source` of the mount.
  * For named volumes, this is the name of the volume. 
  * For anonymous volumes, this field is omitted. May be specified as `source` or `src`.
  * For `bind mounts`, this is the path to the file or directory on the Docker daemon host. May be specified as source or src.
* The `destination` takes as its value the path where the file or directory is mounted in the container. May be specified as `destination`, `dst`, or `target`.
  
* The `readonly` option, if present, causes the bind mount to be [mounted into the container as read-only]([mounted](https://docs.docker.com/storage/volumes/#use-a-read-only-volume)). May be specified as `readonly` or `ro`.
* volumes options (work with volumes)
  * The `volume-opt` option, which can be specified more than once, takes a key-value pair consisting of the option name and its value.
* bind mounts options (work with bind mounts)
  * The `bind-propagation` option, if present, changes the bind propagation. May be one of `rprivate`, `private`, `rshared`, `shared`, `rslave`, `slave`


## Good use cases for volumes/bind mounts

### For volumes
* Sharing data among multiple running containers.
* When you want to store your container’s data on a remote host or a cloud provider, rather than locally.（volume driver）
* When you need to back up, restore, or migrate data from one Docker host to another, volumes are a better choice.
* When your application requires __high-performance I/O__ on Docker Desktop. Volumes are stored in the Linux VM rather than the host, which means that the reads and writes have much lower latency and higher throughput.
* 使用 `volume`可以屏蔽掉多os系统文件系统的差异，统一有`docker volume`来管理

### for bind mounts

* Sharing configuration files from the host machine to containers. 比如解决时区、dns问题等 
* When the file or directory structure of the Docker host is guaranteed to be consistent with the bind mounts the containers require.


## 结论
对于`volume`与`bind mounts`通过上文的讨论，你应该知道如何选择，确实`volume`看起来是一个更完整的设计，也推荐你使用；你可以在`docker run`或是`docker-compose`中好好享受`volume`，但是在`dockerfile`不建议使用`volume`来声明volume, 更多地可以看[这里的说明](https://github.com/warm-native/docs/blob/master/docker/dockerfile-guide.md#volume)


## Reference to
* [docker storage](https://docs.docker.com/storage/)
* [dockerfile guide](https://github.com/warm-native/docs/blob/master/docker/dockerfile-guide.md)

![云原生运维实践-mp](https://user-images.githubusercontent.com/5203608/166955094-df7958d2-4327-4c57-b2fd-ed4f2c25d4e1.png)