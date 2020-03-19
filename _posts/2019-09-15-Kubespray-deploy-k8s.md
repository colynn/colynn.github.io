---
layout: post
title: Kubespary 部署 Kubernetes 1.5.3 版本
tags: [kubernetes]
comments: true
---
## 

### 前置条件
1. 熟悉linux基本命令
2. 了解k8s基本概念
3. K8s master推荐配置

  节点数量 | 推荐配置（vCPU/Memory/磁盘）
  ------------- | -------------
  1-5 nodes | 1C/4G/4G
  6-10 nodes | 2C/8G/32G
  11-100 nodes | 4C/16G/80G
  101-250 nodes | 8C/32G/160G
  251-500 nodes | 16C/32G/200G
  大于500 nodes | 32C/64G/200G


### 安装环境
名称| 内网IP | 配置 
------ | ------| ------|
node0  | 10.0.0.182 | 2c*4G |
node1  | 10.0.0.183 | 2c*4G |

## 集群安装步骤
### 1. 设置master与node1间的免密登录
由于kubespray是依赖于ansible，ansible通过ssh协议进行主机之间的访问，所以部署之前需要设置主机之间免密登录，步骤如下：

```
[node0]$ ssh-keygen -t rsa
[node0]$ scp ~/.ssh/id_rsa.pub root@node1:/root/.ssh
[local terminal]$ ssh root@node1
[node1]$ cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
```

### 2. 下载kubespray

_注_：此步骤工作环境为node0
```
$ wget https://github.com/kubernetes-sigs/kubespray/archive/v2.11.0.tar.gz
$ tar -zxvf v2.11.0.tar.gz
$ cd kubespray-v2.11.0
```

### 3. 配置调整

#### 3.1 更换镜像
Kubernetes安装大部分都是使用的国外的镜像，由于防火墙原因没有办法获取到这些镜像，所以需要自己创建镜像仓库并将这些镜像获取到上传到镜像仓库中。

__注__: 可以使用自建的镜像仓库也可以使用国内的镜像仓库, 具体仓库选择查看附录1.

_注_: 如何使用国内的镜像仓库, 可以略过以下 子步骤。
* 新建镜像仓库

  _注_: 如何使用国内的镜像仓库, 可以略过此步骤。

  镜像仓库我们选用的组件是Harbor，
  安装步骤参考：[https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md](https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md)

* 整理k8s集群部署中需要使用的镜像
在文件roles/download/defaults/main.yml文件中，可以看到使用的全量镜像列表，注意某些镜像暂时没有用到.
_注_: 主要涉及镜像有 kubernetes-*; calico_*_image_repo; coredns; etcd

* 下载所需镜像并上传至私有镜像仓库

使用的镜像列表如下，在这里我申请了一台国外的阿里云主机，在该台主机下载所需镜像然后上传至私有镜像仓库
可以将使用的镜像列表，参照附录1替换为国内站点，然后本地下载后，上传至私有镜像仓库。

示例操作如下：

```
docker pull gcr.azk8s.cn/google_containers/kubernetes-dashboard-amd64:v1.10.0
docker tag gcr.azk8s.cn/google_containers/kubernetes-dashboard-amd64:v1.10.0 [harbor-ip]:5000/google_containers/kubernetes-dashboard-amd64:v1.10.0
docker push [harbor-ip]:5000/google_containers/kubernetes-dashboard-amd64:v1.10.0
```
_注_: [harbor-ip] 是为私有的镜像仓库地址。

* 更改部署脚本的镜像地址
  1. 生成部署的参数文件
  
  ```
  [node0 kubespray-v2.11.0]$  cp -rfp inventory/sample inventory/[your-cluster-name]
  ```
  
  2. 替换为新的镜像地址
  
  ```
  [node0 kubespray-v2.11.0]$ vim inventory/[your-cluster-name]/group_vars/k8s-cluster/k8s-cluster.yml

  # kubernetes image repo define
  kube_image_repo: "[harbor-info]/google_containers"

  # comment: 将使用的组件的镜像仓库修改为私有镜像仓库地址
  etcd_image_repo: "[harbor-info]/coreos/etcd"
  coredns_image_repo: "[harbor-info]/coredns"

  calicoctl_image_repo: "[harbor-info]/calico/ctl"
  calico_node_image_repo: "[harbor-info]/calico/node"
  calico_cni_image_repo: "[harbor-info]/calico/cni"
  calico_policy_image_repo: "[harbor-info]/calico/kube-controllers"
  ```


_注_: 如果[harbor-info]为非https的站点，需要在 `inventory/[your-cluster-name]/group_vars/all/docker.yml`文件中添加如下配置：

```
docker_insecure_registries:
   - [harbro-ip]:[port]
```


#### 3.2 Docker安装源更改

由于默认从Docker官方源安装docker，速度非常慢，这里我们更换为国内阿里源，在`inventory/[your-cluster-name]/group_vars/k8s-cluster/k8s-cluster.yml`文件中添加如下配置：

```
# CentOS/RedHat docker-ce repo
docker_rh_repo_base_url: 'https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/stable'
docker_rh_repo_gpgkey: 'https://mirrors.aliyun.com/docker-ce/linux/centos/gpg'
dockerproject_rh_repo_base_url: 'https://mirrors.aliyun.com/docker-engine/yum/repo/main/centos/7'
dockerproject_rh_repo_gpgkey: 'https://mirrors.aliyun.com/docker-engine/yum/gpg'
```

#### 3.3 可执行文件预处理
_注_: 此步骤可根据的网络环境决定是否需要略过

另外由于需要从google以及github下载一些可执行文件，由于防火墙原因无法直接在服务器上下载，我们可以预先将这些执行文件下载好，然后上传到指定的服务器路径中

可执行文件下载地址可以在`roles/download/defaults/main.yml`文件中查找到，下载路径如下：

```
kubeadm_download_url: "https://storage.googleapis.com/kubernetes-release/release/v1.12.5/bin/linux/amd64/kubeadm"
hyperkube_download_url: "https://storage.googleapis.com/kubernetes-release/release/v1.12.5/bin/linux/amd64/hyperkube"
cni_download_url: "https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz"
```

接下来修改文件权限，并上传到每台服务器的`/tmp/releases`目录下

```
chmod 755 cni-plugins-amd64-v0.6.0.tgz hyperkube kubeadm
scp cni-plugins-amd64-v0.6.0.tgz hyperkube kubeadm root@node1:/tmp/releases
```

### 4. 执行安装命令

```
# Install dependencies from ``requirements.txt``
sudo pip install -r requirements.txt

# Update Ansible inventory file with inventory builder
declare -a IPS=(10.0.0.182 10.0.0.183)
CONFIG_FILE=inventory/[your-cluster-name]/hosts.ini python3 contrib/inventory_builder/inventory.py ${IPS[@]}

# Review and change parameters under ``inventory/[your-cluster-name]/group_vars``
cat inventory/mycluster/group_vars/all/all.yml
cat inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml

# Deploy Kubespray with Ansible Playbook - run the playbook as root
# The option `-b` is required, as for example writing SSL keys in /etc/,
# installing packages and interacting with various systemd daemons.
# Without -b the playbook will fail to run!
ansible-playbook -i inventory/[your-cluster-name]/hosts.ini --become --become-user=root cluster.yml
```
### 5. 确认集群安装状态

```

$ kubectl cluster-info

$ kubectl get pods --all-namespaces

```

## 常用问题
### 问题1. coreDns 有一个节点未能正常启动

__注__: 如果是单节点部署K8s，Kubespray默认会创建2个coredns Pod，但Deployment中又用到了podAntiAffinity，因此会导致其中一个coredns pod pending，所以需要修改代码如下

```
vim ./roles/kubernetes-apps/ansible/templates/coredns-deployment.yml.j2
```

//注释掉以下几行代码

```
    affinity:
    #podAntiAffinity:
    #  requiredDuringSchedulingIgnoredDuringExecution:
    #  - topologyKey: "kubernetes.io/hostname"
    #    labelSelector:
    #      matchLabels:
    #        k8s-app: coredns{{ coredns_ordinal_suffix | default('') }}
```
或者在spec一行添加代码：

```
spec:

  replicas: 1   //指定pod为1个副本
```

### 问题2. 异常【kubelet cgroup driver：cgroupfs跟docker cgroup driver：systemd不一致】

* 异常描述 

```
error: failed to run Kubelet: failed to create kubelet: misconfiguration: kubelet cgroup driver: “systemd” is different from docker cgroup driver: "cgroupfs"
```

* 解决方式
  方式1. 修改docker
  方式2. 修改kubelet配置

  ```
  $ sed -i 's/cgroupDriver: systemd/cgroupDriver: cgroupfs/'   /etc/kubernetes/kubelet-config.yaml
  $ sed -n '/cgroupDriver:/p' /etc/kubernetes/kubelet-config.yaml  ## 确认修改的配置

  $ systemctl restart kubelet
  ```


## 附录1 国内镜像仓库

1. 常用镜像仓库；

* DockerHub镜像仓库: https://hub.docker.com/
* 阿里云镜像仓库： https://cr.console.aliyun.com
* google镜像仓库：
https://console.cloud.google.com/gcr/images/google-containers/GLOBAL
* coreos镜像仓库：
https://quay.io/repository/
* RedHat镜像仓库：
https://access.redhat.com/containers

部分国外镜像仓库无法访问，但国内有对应镜像源，例如kubernetes相关镜像、coreos相关镜像国内无法直接拉取，可以从以下镜像源拉取：

2. 示例：微软google gcr镜像源

```
#以gcr镜像为例，以下镜像无法直接拉取
docker pull gcr.io/google-containers/kube-apiserver:v1.15.2
#改为以下方式即可成功拉取：
docker pull gcr.azk8s.cn/google-containers/kube-apiserver:v1.15.2
```

微软coreos quay镜像源

```
#以coreos镜像为例，以下镜像无法直接拉取
docker pull quay.io/coreos/kube-state-metrics:v1.7.2
#改为以下方式即可成功拉取：
docker pull quay.azk8s.cn/coreos/kube-state-metrics:v1.7.2
```

微软dockerhub镜像源

```
#以下方式拉取镜像较慢
docker pull centos
#改为以下方式使用微软镜像源：
docker pull dockerhub.azk8s.cn/library/centos
docker pull dockerhub.azk8s.cn/willdockerhub/centos
```

dockerhub google镜像源

```
#以gcr镜像为例，以下镜像无法直接拉取
docker pull gcr.io/google-containers/kube-apiserver:v1.15.2
#改为以下方式即可成功拉取：
docker pull mirrorgooglecontainers/google-containers/kube-apiserver:v1.15.2
```

阿里云google镜像源

```
#以gcr镜像为例，以下镜像无法直接拉取
docker pull gcr.io/google-containers/kube-apiserver:v1.15.2
#改为以下方式即可成功拉取：
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.15.2
————————————————
```



### 参考文档
1. https://blog.csdn.net/networken/article/details/84571373
2. https://jicki.me/kubernetes/docker/2018/12/21/k8s-1.13.1-kubespray/


