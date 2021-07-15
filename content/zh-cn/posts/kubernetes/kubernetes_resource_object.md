---
title: "Kubernetes对象详解"
categories: ["技术博客"]
tags: ["kubernetes", "DevOps"]
author: "Jack Wu"
date: 2020-10-26T10:54:43+08:00
draft: false
---

## Kubernetes对象的概念
在 Kubernetes 系统中，Kubernetes 对象 是持久化的实体。 Kubernetes 使用这些实体去表示整个集群的状态。它们主要描述了如下信息：

	哪些容器化应用在运行（以及在哪些节点上）

	可以被应用使用的资源

	关于应用运行时表现的策略，比如重启策略、升级策略，以及容错策略
## kubernetes对象的类别

### 资源对象的类别

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


### 配置对象

Node、Namespace、Service、Secret、ConfigMap、Ingress、Label、ThirdPartyResource、 ServiceAccount

### 存储对象

#### Volume

	volume是kubernetes为防止容器重新创建启动时导致的数据丢失、解决容器之间的文件数据共享问题而抽象出的对象。

#### Persistent Volume
	
	持久卷（PersistentVolume，PV）是集群中的一块存储，可以由管理员事先供应，或者 使用存储类（Storage Class）来动态供应。 持久卷是集群资源，就像节点也是集群资源一样。PV 持久卷和普通的 Volume 一样，也是使用 卷插件来实现的，只是它们拥有独立于任何使用 PV 的 Pod 的生命周期。

### 策略对象

SecurityContext、ResourceQuota、LimitRange

## Kubernetes对象的描述

必须字段：

apiVersion - 创建该对象所使用的 Kubernetes API 的版本

kind - 想要创建的对象的类别

metadata - 帮助唯一性标识对象的一些数据，包括一个 name 字符串、UID 和可选的 namespace

非必须字段：

spec - 关于对象的更为详细的描述	

### 示例

  apiVersion: v1 #指定api版本，此值必须在kubectl apiversion中   
  kind: Pod #指定创建资源的角色/类型   
  metadata: #资源的元数据/属性   
    name: test-pod #资源的名字，在同一个namespace中必须唯一   
    labels: #设定资源的标签 
      k8s-app: apache   
      version: v1   
      kubernetes.io/cluster-service: "true"   
    annotations:            #自定义注解列表   
      - name: String        #自定义注解名字   
  spec: #specification of the resource content 指定该资源的内容   
    restartPolicy: Always #表明该容器一直运行，默认k8s的策略，在此容器退出后，会立即创建一个相同的容器   
    nodeSelector:     #节点选择，先给主机打标签kubectl label nodes kube-node1 zone=node1   
      zone: node1   
    containers:   
    - name: test-pod #容器的名字   
      image: 10.192.21.18:5000/test/chat:latest #容器使用的镜像地址   
      imagePullPolicy: Never #三个选择Always、Never、IfNotPresent，每次启动时检查和更新（从registery）images的策略， 
                            # Always，每次都检查 
                            # Never，每次都不检查（不管本地是否有） 
                            # IfNotPresent，如果本地有就不检查，如果没有就拉取 
      command: ['sh'] #启动容器的运行命令，将覆盖容器中的Entrypoint,对应Dockefile中的ENTRYPOINT   
      args: ["$(str)"] #启动容器的命令参数，对应Dockerfile中CMD参数   
      env: #指定容器中的环境变量   
      - name: str #变量的名字   
        value: "/etc/run.sh" #变量的值   
      resources: #资源管理 
        requests: #容器运行时，最低资源需求，也就是说最少需要多少资源容器才能正常运行   
          cpu: 0.1 #CPU资源（核数），两种方式，浮点数或者是整数+m，0.1=100m，最少值为0.001核（1m） 
          memory: 32Mi #内存使用量   
        limits: #资源限制   
          cpu: 0.5   
          memory: 1000Mi   
      ports:   
      - containerPort: 80 #容器开发对外的端口 
        name: httpd  #名称 
        protocol: TCP   
      livenessProbe: #pod内容器健康检查的设置 
        httpGet: #通过httpget检查健康，返回200-399之间，则认为容器正常   
          path: / #URI地址   
          port: 80   
          #host: 127.0.0.1 #主机地址   
          scheme: HTTP   
        initialDelaySeconds: 180 #表明第一次检测在容器启动后多长时间后开始   
        timeoutSeconds: 5 #检测的超时时间   
        periodSeconds: 15  #检查间隔时间   
        #也可以用这种方法   
        #exec: 执行命令的方法进行监测，如果其退出码不为0，则认为容器正常   
        #  command:   
        #    - cat   
        #    - /tmp/health   
        #也可以用这种方法   
        #tcpSocket: //通过tcpSocket检查健康    
        #  port: number    
      lifecycle: #生命周期管理   
        postStart: #容器运行之前运行的任务   
          exec:   
            command:   
              - 'sh'   
              - 'yum upgrade -y'   
        preStop:#容器关闭之前运行的任务   
          exec:   
            command: ['service httpd stop']   
      volumeMounts:  #挂载持久存储卷 
      - name: volume #挂载设备的名字，与volumes[*].name 需要对应     
        mountPath: /data #挂载到容器的某个路径下   
        readOnly: True   
    volumes: #定义一组挂载设备   
    - name: volume #定义一个挂载设备的名字   
      #meptyDir: {}   
      hostPath:   
        path: /opt #挂载设备类型为hostPath，路径为宿主机下的/opt,这里设备类型支持很多种 
      #nfs，