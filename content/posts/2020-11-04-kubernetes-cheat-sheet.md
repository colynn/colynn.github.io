---
title: K8s常用命令备注手册
categories: ["教程"]
tags: ["运维", "kubernetes"]
date: 2020-11-04
lastmod: 2020-11-04
---

## Pod

1. pod 进入指定的 container

```sh
# Name of the container: kaniko
# Name of the pod: jnlp-gjrpc
$ kubectl exec -ti jnlp-gjrpc -c kaniko /bin/sh
```

2. 强制删除 pod

```sh
$ kubectl delete pods mypod-ba97bc8ef-8rgaa --grace-period=0 --force
```

## Label

1. create a label for a node:

```sh
$ kubectl label node <nodename> <labelname>=<value>
```

2. delete above labels from its node

```sh
$ kubectl label node <nodename> <labelname>-
```

3. confirm node labels

```sh
$ kubectl get nodes --show-labels
```