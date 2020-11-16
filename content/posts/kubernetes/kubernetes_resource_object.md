---
title: "Kubernetes资源对象详解"
tags: ["kubernetes", "DevOps"]
author: "Jack Wu"
date: 2020-10-26T10:54:43+08:00
draft: false
---

## Kubernetes 资源对象

#### Pod
Pod 是最小的可部署的 Kubernetes 对象模型。Pod 表示集群上正在运行的进程。一个 Pod 由一个或多个容器组成，Pod 中容器共享存储和网络，在同一台 Docker 主机上运行。在 kubernetes 中，若要运行一个容器，则必须先创建 pod，让容器在 pod 中运行，可以把 Pod 看成是容器的运行环境。
  Docker 是 Kubernetes Pod 中最常用的容器运行时，但 Pod 也能支持其他的容器运行时，如 rtk。
  Kubernetes 集群中的 Pod 可被用于以下两个主要用途：

运行单个容器的 Pod。”每个 Pod 一个容器”模型是最常见的 Kubernetes 用例；在这种情况下，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod，而不是容器。

运行多个协同工作的容器的 Pod。 Pod 可能封装由多个紧密耦合且需要共享资源的共处容器组成的应用程序。 这些位于同一位置的容器可能形成单个内聚的服务单元——一个容器将文件从共享卷提供给公众，而另一个单独的“挂斗”容器则刷新或更新这些文件。 Pod 将这些容器和存储资源打包为一个可管理的实体。

#### ReplicationController
  ReplicationController 简写 “RC” 或 “RCS”。译作“副本控制器”，“Replication” 就是“复制”、“副本”的意思。ReplicationController 确保在任何时候都有特定数量的 pod 副本处于运行状态。 换句话说，ReplicationController 确保一个 pod 或一组同类的 pod 总是可用的。
  当 pods 数量过多时，ReplicationController 会终止多余的 pods。当 pods 数量太少时，ReplicationController 将会启动新的 pods。 与手动创建的 pod 不同，由 ReplicationController 创建的 pods 在失败、被删除或被终止时会被自动替换。

#### ReplicaSet
  ReplicaSet 简写 “RS”，是 “Replication Controller” 的升级版。和 “ReplicationController” 一样用于确保任何给定时间指定的Pod副本数量，并提供声明式更新等功能。
RC与RS唯一区别就是lable selectore支持不同，RS支持新的基于集合的标签，RC仅支持基于等式的标签。
#### Deployment
  Deployment是一个更高层次的API对象，他管理ReplicaSets和Pod，并提供声明式更新等功能。
官方建议使用Deployment管理ReplicaSets，而不是直接使用ReplicaSets，这就意味着可能永远不需要直接操作ReplicaSet对象。

#### StatefulSet
  StatefulSet适合持久性的应用程序，有唯一的网络标识符（IP），持久存储，有序的部署、扩展、删除和滚动更新。

#### DaemonSet
  Daemon 一词应该不陌生的，Daemon 进程（守护进程）、Demon 程序（守护程序）。顾名思义，DaemonSet 一般是用来部署一些特殊应用的，譬如日志应用等有“守护”意义的应用。
  DaemonSet确保所有（或一些）节点运行同一个Pod（当然这里不是指“同一个”，而是和副本一样的概念，当然不可能多个节点运行“同一个”Pod，它又不是量子态）。当节点加入kubernetes集群中，Pod会被调度到该节点上运行，当节点从集群中移除时，DaemonSet的Pod会被删除。删除DaemonSet会清理它所有创建的Pod。

#### Job
  一次性任务，运行完成后Pod销毁，不再重新启动新容器。

#### CronJob
  CronJob 是在 Job 基础上加上了定时功能。

#### HorizontalPodAutoscaling
Horizontal Pod Autoscaling可以根据CPU使用率或应用自定义metrics自动扩展Pod数量（支持replication controller、deployment和replica set）。

控制管理器每隔30s（可以通过–horizontal-pod-autoscaler-sync-period修改）查询metrics的资源使用情况

支持三种metrics类型

预定义metrics（比如Pod的CPU）以利用率的方式计算

自定义的Pod metrics，以原始值（raw value）的方式计算

自定义的object metrics

支持两种metrics查询方式：Heapster和自定义的REST API

支持多metrics

自定义metrics

  使用方法

  控制管理器开启–horizontal-pod-autoscaler-use-rest-clients

  控制管理器的–apiserver指向API Server Aggregator

  在API Server Aggregator中注册自定义的metrics API


目前K8s中的业务主要可以分为长期伺服型（long-running）、批处理型（batch）、节点后台支撑型（node-daemon）和有状态应用型（stateful application）；分别对应的小机器人控制器为Deployment、Job、DaemonSet和PetSet，