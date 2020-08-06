---
layout: post
title: kubernetes-网络组件-flannel-calico
categories: ["教程"]
tags: ["kubernetes", "flannel", "calico"]
date: 2020-04-10
lastmod: 2020-04-10
---

# 前言

网络是kubernetes核心组件， 但是理解它具体是怎样工作却有一定挑战性，主要有4个不同的网络问题要解决：
1. 高度耦合的容器到容器通信：这可以通过[pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/)和localhost通信解决。
2. Pod与Pod之间的通信；
3. Pod与Service间的通信, [services](https://kubernetes.io/docs/concepts/services-networking/service/)；
4. 外部和 Service间的通信，[services](https://kubernetes.io/docs/concepts/services-networking/service/)


无论是哪种网络插件，kubernets都要求其满足如下的基础要求。
1. 主机上的pod不需要NAT可以直接与所有主机上的所有pods通信；
2. 主机上的进程（系统进程，kubelet）可以与该主机上的所有pod通信；
3. 依赖主机网络的主机上运行的pods不通过NAT可以与所有主机上的所有pod通信。（注意：对于那些支持在主机网络中运行Pod的平台（例如Linux））

该模型不仅总体上不那么复杂，而且基本上与Kubernetes的愿望兼容，以实现将应用程序从VM廉价迁移到容器。就像之前应用在VM中运行的则您的VM具有IP，并且可以与项目中的其他VM通信是一样的模型。

Kubernetes不提供任何默认的网络实现，而是仅定义模型并由其他工具来实现，下文作者跟大家分享flannel及calico组件。

# flannel

## what is flannel
Flannel是CoreOS专门为kubernetes设计3层网络结构的简单方法。为了实现kubernetes的网络要求，flannel 在主机网络之上创建了平面网络（也就是overlay 网络）。在这个网络下，为会每一个主机配置一个ip及子网范围，且主机上所有的容器（Pod）将会有一个IP地址，flannel使用etcd维护子网与主机间的关系，基于kernel路由表传输IP数通包给其他主机上的容器。

__注__: 使用UDP作为传输协议是因为其更容易通过防火墙。


## how it work
[基于 Flannel 查看跨主机容器通信](https://colynn.github.io/2020-04-02-vxlan-is-what/)


## 小结
Flannel对于大多数用户来说是个不错的选择。从管理的角度来看，它提供了一个简单的网络模型，当你只需要基本的东西时，它提供了一个简单的网络模型，可以建立一个适合大多数用例的环境。总的来说，在你需要它所不能提供的东西之前(比如性能、网络策略等)，从Flannel开始使用是一个安全的选择。


# calico
Calico 是适用于容器、虚拟机各裸机工作负载的开源网络和网络安全解决方案。Calico 使用标准Linux 网络工具为云原生应用程序提供两项主要服务：
    * 工作负载之前的网络连接。
    * 在工作负载之间实施网络安全策略。

不像 Flannel, Calico 不使用 overlay 网络，而是使用 [BGP 路由协议](https://en.wikipedia.org/wiki/Border_Gateway_Protocol)配置一个3层网络在主机之间路由数据包。这意味着在主机之间移动时，数据包不需要包装在额外的封装层中。 BGP路由机制可以直接引导数据包，而无需执行将流量包装在附加流量层中的额外步骤。

除了带来了更好的网络性能，也有一个好的点，当出现网络问题时它允许进行更常规的故障排除。 尽管使用VXLAN之类的技术封装的解决方案效果很好，但该过程处理数据包的方式可能会使跟踪变得困难。而使用Calico，标准的调试工具可以访问与简单环境中相同的信息，从而使更广泛的开发人员和管理员更容易理解如何排错。

对于[网络策略](https://kubernetes.io/docs/concepts/services-networking/network-policies/)也是 Calico 很受欢迎的功能之一。Calico还可以与服务网状网Istio集成，以便在服务网状层和网络基础架构层解释和实施群集内工作负载的策略。 这意味着您可以配置功能强大的规则，描述Pod应如何发送和接受流量，从而提高网络环境的安全性和可控性。

## how it work

[Calico Route](https://medium.com/@jain.sm/flannel-vs-calico-a-battle-of-l2-vs-l3-based-networking-5a30cd0a3ebd)

## 小结
当你的环境如果需要一个好的性能，网络策略时Calico是一个好的选择， 此外，Calico提供商业支持。


## 参考
1. [Kubernetes: Flannel networking](https://blog.laputa.io/kubernetes-flannel-networking-6a1cb1f8ec7c)
2. [Comparing Kubernetes CNI Providers](https://rancher.com/blog/2019/2019-03-21-comparing-kubernetes-cni-providers-flannel-calico-canal-and-weave/)

