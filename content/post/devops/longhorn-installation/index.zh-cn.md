---
title: "Longhorn 安装与反安装"
description: 
date: 2024-03-19T11:50:30+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
categories: ['Devops']
tags: ['k8s','longhorn']
---

## 首先最重要的事情

1. 好好阅读官方文档
2. 读完官方文档
3. 动手之前先三思

## 过程



我本想测试一下效果，因此直接通过 terraform 的 helm provider 做了安装，创建了一个新的 namespace 叫做 `longhorn` ，默认没有设置 values。这就是我犯的**第 1 个错**。 namespace 没有使用官方推荐的 `longhorn-system`。

```
resource "helm_release" "longhorn" {
  name = "longhorn"
  namespace = "longhorn"
  create_namespace = true
  repository = "https://charts.longhorn.io"
  chart      = "longhorn"
  version    = "1.6.0"
}

```

官方推荐使用这样的 namespace 是有寓意的，因为后面很多操作和教程都是依赖于这个假设和前提，如果自己用了别的，那么意味着很多脚本和操作包括 deployment 都得自己调整 namespace，实际上这些时间本来都不必耗费。

安装完成之后我发现集群资源不够，通过排查

于是删除这段代码重新 apply，但是发现存在报错。下层的原理是 terraform 会调用 helm uninstall 进行删除，我发现
