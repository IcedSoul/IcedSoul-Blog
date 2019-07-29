title: 使用Kubeadm搭建Kubernetes 1.15 集群
tags:
- Kubernetes
- Kubeadm
- 集群搭建
categories: Kubernetes
---

## 前言

在上一篇博客中介绍了Kubernetes的一些概念和一些常见术语，读过之后大家应该对Kubernetes的定位和大概组织结构有了一些基本的认识，为了接下来进一步进行实验和学习，这节课我们先来搭建一个基本的Kubernetes集群，后续可以依托这个集群来进一步学习其内容。

Kubernetes的安装方式有很多种，可以选择安装一个基本可用的MiniKube，也可以使用Kubeadm来安装一个常见的Kubernetes集群，还可以使用二进制安装的方式安装一个用于生产环境的集群。

在这篇博客中，我们选择使用Kubeadm工具来安装一个最常见的k8s集群。目前为止，k8s的最新版本是1.15，所以在这篇博客中我们选择安装1.15版本，如果你看到博客时k8s已经有了更新的版本，那么你可以安装当前最新版本进行安装学习，相信安装方式不会产生太大的变化，如果有较大不同我会在更新此篇博客。

## 准备工作

安装k8s集群需要准备以下内容：

1. 至少两台服务器最好是3台以上，默认使用CentOS系统。（可以在本地安装几台CentOS虚拟机，如果服务器是其它Linux发行版，可参考官网相关安装方式）
2. 每台服务器配置最好为2核4G。（如果配置较低也不是不行，可能会出问题。不过你也可以试着搭一搭，毕竟只是为了学习的话一般也不会跑太大的系统。）
3. 保证所有服务器网络畅通。（可以互相ping通。）

如果只有一台服务器，可以进行k8s官网安装Minikube，或者使用Kubeadm按照安装Master的方式来安装。



## 参考资料

1. 《Kubernetes 权威指南 第四版 从Docker到Kubernetes实践全接触》 龚正 等 中国工信出版社
2. 《每天五分钟 玩转Kubernetes》 CloudMan 著 清华大学出版社
3. Kubernetes官方安装教程： https://kubernetes.io/zh/
4. Kubernetes中文社区安装教程博客：