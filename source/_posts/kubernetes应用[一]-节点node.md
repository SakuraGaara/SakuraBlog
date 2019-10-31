---
title: kubernetes应用[一]-节点node 
categories: kubernetes  
tags:  
  - kubernetes  
---

> Node 是 Pod 真正运行的主机，可以是物理机，也可以是虚拟机。为了管理 Pod，每个 Node 节点上至少要运行 container runtime（比如 docker 或者 rkt）、kubelet 和 kube-proxy 服务。

- 查看node节点
```sh
kubectl describe node
```
每个Node都包括以下状态信息
地址：包括hostname、外网IP和内网IP
条件（Condition）：包括OutOfDisk、Ready、MemoryPressure和DiskPressure
容量（Capacity）：Node上的可用资源，包括CPU、内存和Pod总数
基本信息（Info）：包括内核版本、容器引擎版本、OS类型等

