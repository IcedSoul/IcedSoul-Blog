title: 使用Kubeadm搭建Kubernetes 1.15 集群
tags:
- Kubernetes
- Kubeadm
- 集群搭建
categories: Kubernetes
---

## 前言

在上一篇博客中我们介绍了Kubernetes的一些概念和一些常见术语，读过之后大家应该对Kubernetes的定位和大概组织结构有了一些基本的认识，为了接下来进一步进行实验和学习，这节课我们先来搭建一个基本的Kubernetes集群，后续可以依托这个集群来进一步学习其内容。

Kubernetes的安装方式有很多种，可以选择安装一个基本可用的MiniKube，也可以使用Kubeadm来安装一个常见的Kubernetes集群，还可以使用二进制安装的方式安装一个用于生产环境的集群。

在这篇博客中，我们选择使用Kubeadm工具来安装一个最常见的k8s集群。目前为止，k8s的最新版本是1.15，所以在这篇博客中我们选择安装1.15版本，如果你看到博客时k8s已经有了更新的版本，那么你可以安装当前最新版本进行安装学习，相信安装方式不会产生太大的变化，如果有较大不同我会更新此篇博客。

## 准备工作

安装k8s集群需要准备以下内容：

1. 至少2台服务器最好是3台以上，linux系统。（可以在本地安装几台CentOS虚拟机，如果系统是其它Linux发行版，可参考官网安装方式）
2. 每台服务器配置最低为2核4G。（如果配置较低也不是不行，可能会出问题。不过你也可以试着搭一搭，毕竟只是为了学习的话一般也不会跑太大的系统。）
3. 保证所有服务器网络畅通。（可以互相ping通。）

如果只有一台服务器，可以去k8s官网查看如何安装Minikube。

## 安装Docker

所有服务器都需要先安装Docker，它用于运行k8s的基础设施服务以及我们之后部署的业务。下面是CentOS 7安装Docker的方式。

```shell
# 安装yum源
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
yum makecache fast

# 查看目前的Docker版本
yum list docker-ce.x86_64  --showduplicates |sort -r

# 安装18.09.7 版本
yum install -y --setopt=obsoletes=0 \
  docker-ce-18.09.7-3.el7

# 运行Docker，开机自启动
systemctl start docker
systemctl enable docker
```

我们选择安装的Docker版本为18.09.7，如果安装其它版本可以根据查看到的Docker版本的名称修改安装时的参数来安装对应版本。

在各个服务器都成功安装Docker后，分别运行如下命令检查Docker是否成功安装，版本是否正确。有可能会自动安装较新版本，不过无所谓，只要成功安装即可。

```shell
# 查看Docker是否正在运行，Active(running)表示正常运行
systemctl status docker

# 查看Docker版本
docker --version
```

## 修改hosts

为了便于服务器之间互相通信，我们最好先修改每台服务器的hosts文件，我本次搭建k8s集群一共使用了四台服务器，IP地址分别为

```
10.141.211.176
10.141.211.180
10.141.212.21
10.141.212.138
```

并且，我准备以10.141.212.21为Master，其它三台服务器为Node来搭建k8s集群。因此，我们需要同时修改四台服务器的hosts文件。

```shell
vi /etc/hosts
```

把hosts文件修改为以下内容（注意，请把IP地址替换为你自己服务器的IP地址，名字也可以任取，自己认得即可）：

```tex
10.141.212.21 master
10.141.211.176 node-1
10.141.211.180 node-2
10.141.212.138 node-3
```

修改完hosts文件后可以使用reboot命令重启各服务器，重启后可以看到命令终端root@后的名字变为上面IP地址后面对应的名字说明修改成功，如果没有说明修改无效，需要重新修改/etc/hosts文件。

## 系统配置

首先，我们需要禁用各服务器的防火墙

```shell
systemctl stop firewalld
systemctl disable firewalld
```

然后，需要设置setlinux

```shell
setenforce 0

vi /etc/selinux/config
```

修改文件中对应标签为如下内容：

```properties
SELINUX=disabled
```

为了保险起见，在修改完setlinux之后，最好重启服务器以保证配置生效，避免后面步骤出现错误。

最后，我们需要添加k8s集群的一些网络配置：

```shell
vi /etc/sysctl.d/k8s.conf
```

在文件中粘贴以下内容：

```properties
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```

保存退出后，执行命令以使修改生效：

```shell
modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf
```

## 安装kubeadm，kubelet，kubectl等工具

以下安装命令是官网提供的，但是官网提供的镜像地址被墙了，所以我们将其替换为了阿里云的镜像地址，国内也可以正常拉取镜像，所有服务器都需要执行以下命令。

```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet
```

执行命令之后即可看到kubeadm，kubelet，kubectl工具已被正常安装或更新。

## 初始化Master

接下来，我们来初始化Master节点，因为最开始我们就准备以10.141.211.21为Master节点，所以我们在该节点上执行以下命令来初始化Master节点，注意，以下操作只需要在Master节点执行。

首先，创建一个文件来存放相关的配置：

```shell
vi kubeadm.yaml
```

然后在文件中粘贴以下内容：

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.141.212.21
  bindPort: 6443
nodeRegistration:
  taints:
  - effect: PreferNoSchedule
    key: node-role.kubernetes.io/master
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.15.0
networking:
  podSubnet: 10.244.0.0/16
```

注意，修改第四行的 IP地址为Master的IP，倒数第三行的版本为想要安装的k8s集群的版本，因为我们采用的集群通信方案是flannel,所以最后两行配置为上面所示，如果采用其它方案则需更改为对应配置。

接下来执行初始化语句：

```shell
kubeadm init --config kubeadm.yaml --ignore-preflight-errors=Swap
```

等待一会儿，不出意外的话就能够成功了。控制台最会会打印一个：

```shell
kubeadm join 10.141.212.21:6443 --token 9wa0h0.yks4a7ubn97pirvp \
    --discovery-token-ca-cert-hash sha256:5bcc552c254b6134920eefcebb272f0ae208096784530fdce334a2cb0a96707e
```

后面的hash值可能略有不同，也不用复制我的，不一样的用不了。你需要复制一下打印出来的这个语句，保存起来，等会儿其它节点加入集群需要用上这条语句。

> 当然，在这一步可能会出各种各样的幺蛾子，可能会花式报错。但是正常情况下不会有问题，如果遇到问题去查就行，都会有解决方案。不过有一点需要注意的是，如果这一步安装失败，重新初始化时一定请执行以下语句来重置集群的初始化，然后再重新执行上述语句进行初始化，以避免因重复初始化引起其它错误。
 ```shell
 # 重置集群
 kubeadm reset
 ifconfig cni0 down
 ip link delete cni0
 ifconfig flannel.1 down
 ip link delete flannel.1
 rm -rf /var/lib/cni/
 ```
> 我自己在搭建或者使用集群中的一些问题我也记录了下来，或许有类似的错误，大家或许可以参考一下：   [Kubernetes相关问题记录](https://wiki.icedsoul.cn/?file=008-%E5%BE%AE%E6%9C%8D%E5%8A%A1/001-Kubernetes/003-Kubernetes%E7%9B%B8%E5%85%B3%E9%97%AE%E9%A2%98%E8%AE%B0%E5%BD%95)

完成初始化之后，我们为了让用户获取访问集群权限，需要执行如下语句

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

接下来，我们就可以使用kubectl的相关命令来查看集群状态：

```shell
kubectl get cs
```

可以看到控制台打印出如下内容：

```shell
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok                  
scheduler            Healthy   ok                  
etcd-0               Healthy   {"health":"true"}
```

可以看到集群组件状态都为Healthy，Master节点已经成功初始化。

## 安装Pod Network

接下来我们需要安装通信组件以使集群各节点之间可以进行通信，官网提供了很多种选择，在这里我们选择使用flannel，在Master节点运行以下命令以安装flannel。

```
mkdir -p ~/k8s/
cd ~/k8s
curl -O https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f  kube-flannel.yml
```



## 其它节点加入集群

Mater节点成功初始化之后，其它节点加入集群就非常简单了，只需要再其它节点执行一下刚刚我们保存的join语句就可以了：

```shell
kubeadm join 10.141.212.21:6443 --token 9wa0h0.yks4a7ubn97pirvp \
    --discovery-token-ca-cert-hash sha256:5bcc552c254b6134920eefcebb272f0ae208096784530fdce334a2cb0a96707e
```

在其它节点都执行完join语句后，在Master节点执行

```shell
kubectl get nodes
```

来查看所有node，可能会如下所示：

```shell
NAME     STATUS     ROLES    AGE   VERSION
master   Ready      master   75m   v1.15.1
node-1   Ready      <none>   30m   v1.15.1
node-2   Ready      <none>   30m   v1.15.1
node-3   NotReady   <none>   51s   v1.15.1
```

注意，因为我的node-3刚刚执行完join语句，所以这里状态还是NotReady，只需要等几分钟，再次执行kubectl get nodes即可看到所有node都处于Ready状态，这时，k8s集群已经初步建立起来了。

## 总结

这篇博客主要讲解了使用kubeadm工具在centos系统搭建k8s集群的过程，大家可以同时参考下面的参考资料结合自己的情况灵活使用上述命令，最终完整集群的搭建。有了一个基本可用的k8s集群，后面我们就可以在这个集群的基础上进一步学习k8s的相关内容。

## 参考资料

1. 《Kubernetes 权威指南 第四版 从Docker到Kubernetes实践全接触》 龚正 等 中国工信出版社
2. 《每天五分钟 玩转Kubernetes》 CloudMan 著 清华大学出版社
3. Kubernetes官方安装教程： https://kubernetes.io/zh/
4. Kubernetes中文社区安装教程博客：https://www.kubernetes.org.cn/5551.html
