---
title: "K8s 上 Nextcloud 数据迁移"
description: 
date: 2024-03-30T18:05:22+08:00
image: 
math: 
categories: ['Ops']
tags: ['k8s']
license: 
hidden: false
comments: true
draft: false
---

线上 K8s 集群因为之前的网络故障导致 OpenEBS 存在一些问题，文件系统需要手动 fsck 才能解除 readonly 的情况。
但是考虑到 openebs 多 replicas 的实现上并不是很清真，每个用到 pvc 的容器都会新建 x defaultReplicas 个 pod 用于挂载，pod 数量会暴增且不好管理。
这次准备切换成 Rancher 旗下的 Longhorn 这个分布式存储系统。

之前会考虑使用 openebs 在于 Longhorn 的资源占用太大，每个节点上都会需要起一个 3840m cpu request 的 manager。但实际使用中发现 openebs 组件很多，
细节也非常繁复，配置起来并不方便，也没有 UI 可以管理和对整个集群的状态做把控，这么一衡量其实 Longhorn 的那点资源消耗也是可以接受的。

具体安装 Longhorn 的细节这边不做过多叙述，后续会另外介绍。

/var/lib/kubelet/pods/460f8339-e305-4a9a-8a51-f1929791b9c8/volumes/kubernetes.io~csi/pvc-4ad57ee8-2600-4971-a0db-e7224514f78a/mount
