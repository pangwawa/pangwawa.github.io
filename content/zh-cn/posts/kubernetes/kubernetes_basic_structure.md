---
title: "Kubernetes基础概念与组件架构"
categories: ["技术博客"]
tags: ["kubernetes", "DevOps"]
author: "Jack Wu"
date: 2020-10-09T17:45:53+08:00
draft: false
---

## 什么是Kubernetes 
Kubernetes 是一个可移植的、可扩展的开源平台，用于管理容器化的工作负载和服务，可促进声明式配置和自动化。Kubernetes(k8s)是Google开源的容器集群管理系统（谷歌内部:Borg）在Docker技术的基础上，为容器化的应用提供部署运行、资源调度、服务发现和动态伸缩等一系列完整功能，提高了大规模容器集群管理的便捷性。

## 核心概念与架构
![导图](../../assets/devops/kubernetes_jiagou.png)
#### service
service  一个Service对象是对应用集群的抽象，比如一个Mysql集群，一个微服务的集群，都会被看做一个service

	特点：

		拥有唯一指定的名称(比如mysql-server)。

		拥有一个虚拟IP(Cluster lP、Service IP或VIP）和端口号。

		能够提供某种远程服务能力。

		被映射到提供这种服务能力的一组容器应用上。
#### pod
 k8s中的最小部署单元，不是一个程序/进程，而是一个环境(包括容器、存储、网络ip:port、容器配置)。其中可以运行1个或多个container（docker或其他容器），在一个pod内部的container共享所有资源，包括共享pod的ip:port和磁盘。

pod是临时性的，用完即丢弃的，当pod中的进程结束、node故障，或者资源短缺时，pod会被干掉。基于此，用户很少直接创建一个独立的pods，而会通过k8s中的controller来对pod进行管理。
controller通过pod templates来创建pod，pod template是一个静态模板，创建出来之后的pod就跟模板没有关系了，模板的修改也不会影响现有的pod。

Pod中封装着应用的容器（有的情况下是好几个容器），存储、独立的网络IP，管理容器如何运行的策略选项。Pod代表着部署的一个单位：kubernetes中应用的一个实例，可能由一个或者多个容器组合在一起共享资源。

在Kubrenetes集群中Pod有如下两种使用方式：

一个Pod中运行一个容器。“每个Pod中一个容器”的模式是最常见的用法；在这种使用方式中，你可以把Pod想象成是单个容器的封装，kuberentes管理的是Pod而不是直接管理容器。

在一个Pod中同时运行多个容器。一个Pod中也可以同时封装几个需要紧密耦合互相协作的容器，它们之间共享资源。这些在同一个Pod中的容器可以互相协作成为一个service单位——一个容器共享文件，另一个“sidecar”容器来更新这些文件。Pod将这些容器的存储资源作为一个实体来管理。


#### container
容器是独立运行的一个或一组应用，是镜像运行时的实体。
#### pod 生命周期

需要注意的是pod的生命周期和container的生命周期有一定的联系，但是不能完全混淆一致。pod状态相对来说要简单一些。这里首先列出pod的状态


1、pending：pod已经被系统认可了，但是内部的container还没有创建出来。这里包含调度到node上的时间以及下载镜像的时间，会持续一小段时间。
2、Running：pod已经与node绑定了（调度成功），而且pod中所有的container已经创建出来，至少有一个容器在运行中，或者容器的进程正在启动或者重启状态。--这里需要注意pod虽然已经Running了，但是内部的container不一定完全可用。因此需要进一步检测container的状态。
3、Succeeded：这个状态很少出现，表明pod中的所有container已经成功的terminated了，而且不会再被拉起了。
4、Failed：pod中的所有容器都被terminated，至少一个container是非正常终止的。（退出的时候返回了一个非0的值或者是被系统直接终止）
5、unknown：由于某些原因pod的状态获取不到，有可能是由于通信问题。 一般情况下pod最常见的就是前两种状态。而且当Running的时候，需要进一步关注container的状态。下面就来看下container的状态有哪些：

#### Container生命周期

1、Waiting：启动到运行中间的一个等待状态。
2、Running：运行状态。
3、Terminated：终止状态。 如果没有任何异常的情况下，container应该会从Waiting状态变为Running状态，这时容器可用。

但如果长时间处于Waiting状态，container会有一个字段reason表明它所处的状态和原因，如果这个原因很容易能标识这个容器再也无法启动起来时，例如ContainerCannotRun，整个服务启动就会迅速返回。（这里是一个失败状态返回的特性，不详细阐述）


当一个容器已经运行起来以后，pod和container的状态是如何关联起来的呢，下面给一些举例来更加形象化其中的关系。

### 分层架构
Kubernetes设计理念和功能其实就是一个类似Linux的分层架构
![导图](../../assets/devops/kubernetes_structure.png)
核心层：Kubernetes最核心的功能，对外提供API构建高层的应用，对内提供插件式应用执行环境

应用层：部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS解析等）

管理层：系统度量（如基础设施、容器和网络的度量），自动化（如自动扩展、动态Provision等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy等）

接口层：kubectl命令行工具、客户端SDK以及集群联邦

生态系统：在接口层之上的庞大容器集群管理调度的生态系统，可以划分为两个范畴

   Kubernetes外部：日志、监控、配置管理、CI、CD、Workflow、FaaS、OTS应用、ChatOps等

   Kubernetes内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理等
### 核心组件
#### Kubernetes主要由以下几个核心组件组成：

etcd  保存了整个集群的状态；

apiserver  提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；

controller manager  负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；

scheduler  负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；

kubelet  负责维护容器的生命周期，同时也负责Volume（CVI）和网络（CNI）的管理；

Container runtime  负责镜像管理以及Pod和容器的真正运行（CRI）；

kube-proxy  负责为Service提供cluster内部的服务发现和负载均衡；


#### 除了核心组件，还有一些推荐的Add-ons：

kube-dns负责为整个集群提供DNS服务

Ingress Controller为服务提供外网入口

Heapster提供资源监控

Dashboard提供GUI

Federation提供跨可用区的集群

Fluentd-elasticsearch提供集群日志采集、存储与查询

