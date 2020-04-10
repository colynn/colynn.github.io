---
layout: post
title: 安装 MiniKube
tags: [kubernetes]
comments: true
---

### 前置条件
* mac os 10.13.6
* Virtual Box

#### 确认环境需求
Minikube 要求在 BIOS 中启用 VT-x / AMD-v 虚拟化， 要检查这是否在OSX / macOS上运行，请执行以下操作：

```
sysctl -a | grep machdep.cpu.features | grep VMX
```

如果有输出，那你很棒棒！

#### 配置Brew 镜像源
```
cd "$(brew --repo)"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

```
### 相关软件安装
* kubectl
* docker (for Mac)
* minikube
* virtualbox

1. 安装kubectl
```
$ brew update && brew install kubectl
```

2. minikube(可以直接通过brew来安装)
```
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v0.30.0/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

> 可以从这里获得更多的版本选择 [kuernetes minikube releases](https://github.com/kubernetes/minikube/releases)

3. docker/virtualbox 是通过直接在相应官网下载镜像的方式来安装的.
另外推荐你也可以尝试如下命令，直接通过brew安装。
```
$ brew cask install docker minikube virtualbox
```

### 开始

#### 1. 启动kubernetes 集群
```
$ minikube start
```

命令的输出日志大概如下：
```
Starting local Kubernetes v1.10.0 cluster...
Starting VM...
Downloading Minikube ISO
 170.78 MB / 170.78 MB [============================================] 100.00% 0s
Getting VM IP address...
Moving files into cluster...
Downloading kubeadm v1.10.0
Downloading kubelet v1.10.0
Finished Downloading kubelet v1.10.0
Finished Downloading kubeadm v1.10.0
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.
```

恭喜！Minikube 为你启动了一个虚拟机，并在该虚拟机中运行了 Kubernetes 集群。

#### 2. 查看集群节点信息
```
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   17m   v1.10.0
```


### 附录
1. 阿里云镜像仓库
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 参考
1. Minikube - Kubernetes本地实验环境 https://yq.aliyun.com/articles/221687
2. Kubernetes 环境搭建 - MacOS https://www.jianshu.com/p/74957f08646b
