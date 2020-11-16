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

###  App管理

#### apply
#### annotate
#### autoscale
#### convert
#### diff
#### edit
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
#### auth
#### cp
#### describe
#### exec
#### logs
#### port-forward
#### proxy
#### top

###  集群管理命令

#### api-versions
#### certificate
#### cluster-info
#### cordon
#### drain
#### taint
#### uncordon


###  KUBECTL SETTINGS AND USAGE
#### alpha
#### api-resources
#### completion
#### config
#### explain
#### options
#### plugin
#### version