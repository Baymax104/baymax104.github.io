---
layout: post
title: Kubernetes入门
categories:
- 开发相关
tags:
- Kubernetes
- k8s
---

## 开始

Kubernetes（K8S）是容器集群管理系统，是一个开源的平台，可以实现容器集群的自动化部署、自动扩缩容、维护等功能

k8s核心组件

- Node：节点，一个节点可以是一个物理机或者虚拟机

- Pod：pod是k8s中的最小调度单元，是一个或多个容器的组合，提供容器运行的环境。通常一个pod中只运行一个容器，每个pod运行时会被分配一个集群内的ip

  > Sidecar模式：将一个辅助容器和一个主容器放在一个pod中，辅助容器称为Sidecar，提供一些辅助的功能，如日志收集，服务监控等
{: .prompt-tip }

- Service：将一组相同功能的pod封装成一个服务，提供多个pod的反向代理入口，Service分为内部服务和外部服务

  - 内部服务：用于集群内部使用
  - 外部服务：通过NodePorts暴露端口给外部使用，通常用于开发和测试使用

  > 注意，Node和Pod并不是层层封装的关系，Node可以看做是一个运行Pod的容器，不同的Pod可以被调度到Node中运行，因此一个Service中的Pod可以位于不同的Node中
{: .prompt-info }

- Ingress：用于管理集群外部访问集群内部服务的入口和方式，可实现配置域名、端口转发、负载均衡等功能

- ConfigMap：用于封装应用程序使用的配置信息

- Secret：用于封装应用程序使用的敏感信息

- Volume：用于将持久化资源挂载到本地磁盘或远程存储上

- Deployment：用于管理应用程序的副本数量和更新策略，用于实现容灾和滚动更新

- StatefulSet：管理应用程序的副本和扩缩容，此外提供了每个应用副本的稳定网络ip和持久化存储

  > 可以使用StatefulSet来管理数据库这类有状态应用程序，但通常的最佳实践是将有状态应用程序从集群中分离出来，单独部署
{: .prompt-tip }

## 架构

k8s使用master-worker架构，master节点负责管理整个集群，worker节点负责运行容器完成功能

### Worker节点

worker节点中主要包含三个组件

- kubelet：管理节点中的pod，保证pod的可用性
- kube-proxy：为节点内部的pod提供网络代理和负载均衡服务
- container-runtime：容器运行时，用于运行具体pod，可以使用不同的容器运行时，通常有DokerEngine、Containerd等

### Master节点

worker中主要包含四个组件

- kube-apiserver：负责提供整个集群的API接口服务，所有Master节点组件都通过该组件通信
- Scheduler：监控集群中所有节点的使用情况，负责将pod调度到不同的节点中
- ControllerManager：管理集群中各组件的状态，确保集群中的各个组件可用
- etcd：是一个高可用的键值存储系统，用于存储集群中所有组件的状态信息

## kubectl

kubectl是k8s管理集群的CLI工具