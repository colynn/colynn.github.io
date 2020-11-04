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
kubectl exec -ti jnlp-gjrpc -c kaniko /bin/sh
```
