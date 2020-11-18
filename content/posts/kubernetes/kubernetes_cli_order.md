---
title: "Kubernetes常用kubectl命令总结"
tags: ["kubernetes", "DevOps"]
author: "Jack Wu"
date: 2020-10-13T18:01:51+08:00
draft: false
---

## kubernetes 常用kubectl命令

### 基本命令
#### create
通过文件名或控制台输入，创建资源。 
#### get 
用于获取一个或多个资源的信息，包括 Namespace、 Pod、 Node、  Deployment、  Service、  ReplicaSet

如

kubectl get cs                          # 查看集群状态

kubectl get nodes                       # 查看集群节点信息

kubectl get ns                          # 查看集群命名空间

kubectl get svc -n kube-system          # 查看指定命名空间的服务

kubectl get pod <pod-name> -o wide      # 查看Pod详细信息

kubectl get pod <pod-name> -o yaml      # 以yaml格式查看Pod详细信息

kubectl get pods                        # 查看资源对象，查看所有Pod列表

kubectl get rc,service                  # 查看资源对象，查看rc和service列表

kubectl get pod,svc,ep --show-labels    # 查看pod,svc,ep能及标签信息

kubectl get all --all-namespaces        # 查看所有的命名空间
#### run
#### expose
#### delete
通过文件名、控制台输入、资源名或者label selector删除资源。
###  App管理

#### apply
通过文件名或控制台输入，对资源进行配置
#### annotate
更新资源的注解
#### autoscale
#### convert
#### diff
#### edit
编辑服务端的资源。
#### kustomize
#### label
#### patch
#### replace
#### rollout
#### scale
#### set
#### wait

###  使用应用程序 相关命令

#### attach
连接到一个正在运行的容器。
#### auth
#### cp
#### describe
输出指定的一个/多个资源的详细信息。
#### exec
在容器内运行命令
#### logs
输出pod中一个容器的日志。
#### port-forward
#### proxy
#### top

###  集群管理命令

#### api-versions
输出服务端支持的API版本
#### certificate
#### cluster-info
显示集群信息
#### cordon
#### drain
#### taint
#### uncordon


###  KUBECTL SETTINGS AND USAGE
#### alpha
#### api-resources
#### completion
#### config
修改kubeconfig配置文件。
#### explain
#### options
#### plugin
#### version
输出服务端和客户端的版本信息。