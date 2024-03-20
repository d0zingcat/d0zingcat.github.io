---
title: 'Kubectl 必知必会'
date: 2023-08-14T23:10:35+08:00
description: 
license: 
hidden: false
comments: true
categories: ['Tutorial']
tags: ['kubectl', 'productivity', 'k8s']
draft: true
---

> 利益声明：坚定的云原生信徒，反对一切非云原生的管理模式，认为 21 世纪还不转向云原生等于自寻死路。

# 准备工作

1. 安装 `kubectl`。
2. 可以再阅读一下[Kubernetes in Action](https://www.goodreads.com/book/show/34013922-kubernetes-in-action)，非常好的一本书，可以直接入门。
3. 还有余力的话可以阅读[官方文档](https://kubernetes.io/zh-cn/docs/concepts/overview/)，对 K8S 有一个 High-level 的了解。

# 基本概念

## 名词
- Pod
  - 最小单位是 Pod，Pod 里面可以有一个或多个容器。
  - Pod 里面的容器共享网络，可以通过 localhost 相互访问。
  - Pod 里面的容器共享存储，可以通过 Volume 相互访问。
- Deployment
  - 
- Service
  - 
- Ingress
  - 
- StatefulSet

## 动词
- Apply
- Delete
- Get
- Describe
