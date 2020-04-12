---
layout: post
title: VXLAN是什么及如何在容器间通信
tags: [运维, flannel]
comments: true
---

## 引言

*  服务器虚拟化基于物理网络基础设施提出了更高的要求，一个物理服务器多个VMs并且每一个vm归有一个自己的Meida Access Control(MAC)地址. 在交换式以太网中为了应对在成百上千的VMs中可能存在的通信连接， 需要较大的MAC地址表。

* 在数据中心根据vlan来对VMs分组的话，可能需要上千个VLANs, 但是当前VLAN个数的限制是4094，不能满足这个场景。

* 数据中心通过需要托管多个租户，而且他们之间是独立的网络区域，如果基于专用的基础设施显然是很不经济的，所以网络工程师选择基于共享的网络实现隔离。在这种场景下，一个常用的问题就是每个租户独立地分配MAC和VLAN IDs, 导致物理网络上的这些信息可能会重复。

* 对于使用2层物理基础设施的虚拟化环境，一个重要的要求是要在整个数据中心甚至是数据中心之间进行第2层网络的扩展，以便有效地分配计算、网络和存储资源。 在这样的网络中，使用传统的方法，如Spanning Tree Protocol (STP)来实现无环路拓扑结构，会导致大量的失效链路。

* 最后一种情况是，网络运营商更倾向于利用IP进行物理基础设施的互联互通通过等成本多路径（ECMP）实现多路径可扩展性, 从而避免了失效的链接）。即使在这种环境下，也有需要保留第2层模型，以实现VM间通信。

上面描述的场景导致了对overly network的要求，该overlay用于通过逻辑“隧道”以封装的格式承载来自各个VM的MAC二层流量。 也是『Virtual eXtensible Local Area Network（VXLAN）』。

概述：VXLAN是一种VLAN扩展技术，它将标准的第2层以太网帧封装在IP内，特别是使用互联网编号分配机构(IANA)分配的UDP端口4789封装中的此MAC地址创建了一个隧道，允许您将第2层网段扩展到任何第3层网络。

虚拟可扩展局域网(VXLAN)是一种试图解决与大型云计算部署相关的可扩展性问题的网络虚拟化技术。

> VXLAN is a Layer 2 overlay scheme on a Layer 3 network.


## VXLAN是如何工作的？

我们已经知道VXLAN是在3层网络之上创建虚拟的2层网络，但是具体是工作的呢？我们知道在传统的VLAN模式下，通过网段内的ARP广播及网关（同网段不需要网关），从而可以将数据包传送到目标地址。

同样，VXLAN也存在这样的问题，但想让数据包在3层之上为2层封装显然是不可以的，从而就是在VXLAN的端口（也就是`VTEP`）提供如下功能：
1. ARP 解决方案: 需要响应来自本地服务器的ARP请求而不广播ARP数据包。
2. 目标搜索：需要找到与目标MAC对应的目的地位置。

[![vtep]({{ site.url }}/img/vxlan/VTEP.png)]({{ site.url }}/img/vxlan/VTEP.png)


为了实现`VTEP`的功能，必须有一些机制可以共享服务器的MAC、IP地址、位置, 基于云原生的时代，下面作者就使用Flannel看下VTEP是如何实现的。

Flannel是为一个为容器创建overlay网络的开源工具，经常和kubernetes一起使用。

* 使用Linux内核原生的VXLAN设备用于数据包的封装；
* Flannel守护进程`flanneld`根据内核请求，通过 "L2/L3 MISS "通知机制，动态填充FDB和ARP表。
    * 这种机制原本的名字是『DOVER extensions』[kernerl commits - add DOVE extensions for VXLAN](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=e4f67addf158f98f8197e08974966b18480dc751)
* 通过`etcd`来共享 IP/MAC 信息。


## 跨主机容器通信

基于一个例子更好地理解VXLAN是怎样工作的。

[![how it work]({{ site.url }}/img/vxlan/flannel.png)]({{ site.url }}/img/vxlan/flannel.png)

> 可以网络结构中看到，`VTEP`在容器外部。这意味着容器不知道隧道的存在。 这就是VXLAN能够使容器误以为它们已连接到同一网段的原因。

如上图所示， 假设Node1的container-1需要与Node2的container-3通信， 我们一起看下数据包是如何传输的。

首先，容器container-1通过虚拟以太网接口(`VETP`)类型的接口veth0将数据包(如下图)传送给网桥`docker0`。 连接着网桥的另一个端点就是`flannel0`, 

_注_: Flannel不控制容器如何与主机通信，而仅控制主机之间的流量传输方式。也就是如果网络头目标IP地址用于VXLAN之外的主机，则将帧发送到物理网络接口，flannel不会做任何处理。但是如何确定目标IP是VXLAN之内或是VXLAN之外，可能还要继续研究flannel相关代码才能了解。

[![tcp ip frame]({{ site.url }}/img/vxlan/tcp-ip-frame.png)]({{ site.url }}/img/vxlan/tcp-ip-frame.png)

可能你会疑问，为什么container-1知识container-3的MAC地址，这是通过标准ARP报文完成的，只是ARP报文将通过VTEP隧道化传输。因为在每个主机上，flannel运行一个守护进程（`flanneld`）, flannel守护进程`flanneld`会根据内核请求，通过 ["L2/L3 MISS "通知机制]((https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=e4f67addf158f98f8197e08974966b18480dc751))，动态填充FDB和ARP表, 并且将IP/MAC信息通过`etcd`存储起来。

当`flanneld`看到来自容器的ARP报文时，它将把它发送到VXLAN多播组，这样所有参与VXLAN的VTEP都会看到这个ARP。底层不会看到这个ARP报文，因为它被隧道化了。因此，底层的硬件不会广播任何ARP报文。

`flanneld`与`flannel0`与`eth0`共同构成了VTEP的管道， `VTEP`是另一种称为虚拟隧道端点的Linux网络结构, 它是VXLAN隧道的入口/出口点。容器中的数据包将在这里使用Linux内核原生的VXLAN设备完成数据包的封装， 添加了VXLAN部分:


[![vxlan enc]({{ site.url }}/img/vxlan/vxlan-encapsulation.png)]({{ site.url }}/img/vxlan/vxlan-encapsulation.png)

首先，我们看到包含VXLAN头和UDP头的传输头。 接下来是外部（VXLAN）网络头。 在这里，我们看到源IP地址和目标IP地址分别是Node1和Node2的`eth0`。 

因Node1/Node2均在同一网段，可以直接找到Node2接点（如果Node1与Node2不在同一网段，则会外部VXLAN网络头在不同的网络之间路由，当然这个时候源及目标MAC也就不是Node1与Node2, 因为通过不同的网络进行路由时，源及目标MAC会发生变化）。 数据包到达Node2，该过程就被逆转。 VXLAN帧将进入`flanneld`并被解封装，然后原始帧将通过veth1从网桥移动到container-3。

## 结论

Flannel对于大多数用户来说是个不错的选择。从管理的角度来看，它提供了一个简单的网络模型，当你只需要基本的东西时，它提供了一个简单的网络模型，可以建立一个适合大多数用例的环境。总的来说，在你需要它所不能提供的东西之前(比如性能、网络策略等)，从Flannel开始使用是一个安全的选择。
因为帧的封装/去封装会增加网络堆栈的开销，硬件加速可以减少开销但无法消除。使用诸如[`Calico`](https://projectcalico.org)之类的解决方案可以避免这种开销。

## 参考
1. [RFC7348- Virtual eXtensible Local Area Network (VXLAN)](https://tools.ietf.org/html/rfc7348#section-3.3)
2. [How vxlan works on linux](https://www.slideshare.net/enakai/how-vxlan-works-on-linux)
3. [Understanding Overlay Networks In Cloud Deployments](
https://community.arm.com/developer/tools-software/tools/b/tools-software-ides-blog/posts/understanding-and-deploying-overlay-networks)