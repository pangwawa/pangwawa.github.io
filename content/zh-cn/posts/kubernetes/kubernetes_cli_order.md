---
title: "Kubernetes常用kubectl命令总结"
categories: ["技术博客"]
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
#创建带有终端的pod
kubectl run -i --tty busybox --image=busybox

#启动一个 nginx 实例
kubectl run nginx --image=nginx

#启动多个副本的pod
kubectl run mybusybox --image=busybox --replicas=5
#### expose
#为 nginx RC 创建服务，启用本地 80 端口连接到容器上的 8000 端口
kubectl expose rc nginx --port=80 --target-port=8000

kubectl expose（将资源暴露为新的 Service）
#### delete
通过文件名、控制台输入、资源名或者label selector删除资源。

#删除基于nginx.yaml定义的名称
kubectl delete -f nginx.yaml

#删除基于Pod.yaml定义的名称
kubectl delete -f pod.yaml

#删除基于rc名定义的名称
kubectl delete rc rc名

#删除基于service定义的名称
kubectl delete service service名

#删除所有Pod
kubectl delete pods --all

#根据label删除
kubectl delete pod -l app=flannel -n kube-system

#删除 pod.json 文件中定义的类型和名称的 pod
kubectl delete -f ./pod.json

#删除名为“baz”的 pod 和名为“foo”的 service
kubectl delete pod,service baz foo

#删除具有 name=myLabel 标签的 pod 和 serivce
kubectl delete pods,services -l name=myLabel

#删除具有 name=myLabel 标签的 pod 和 service，包括尚未初始化的
kubectl delete pods,services -l name=myLabel --include-uninitialized

#删除 my-ns namespace下的所有 pod 和 serivce，包括尚未初始化的
kubectl -n my-ns delete po,svc --all

#强制删除
kubectl delete pods prometheus-7fcfcb9f89-qkkf7 --grace-period=0 --force

#卸载kubernetes-dashboard
kubectl delete -f kubernetes-dashboard.yaml
###  App管理

#### apply
通过文件名或控制台输入，对资源进行配置

#创建/更新
kubectl apply -f xxx.yaml
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