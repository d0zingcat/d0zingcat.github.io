---
author: ["Me"]
title: 'K8s on Baremetal Servers'
date: 2024-11-20T16:24:42+08:00
categories: []
tags: []
draft: true
---

> This is a series posts which illustrates how to start a brand new k8s cluster on bare metal servers. I will introduce all the procedure of that and show you how to maintain a production level k8s cluster step by step. 

Here are the posts(Listed in order):

[K8s on Bare metal: Creation(This post)](https://d0zingcat.dev/archives/k8s-on-bare-metal-creation)

## Prerequisites

Hardware requirements for K8s cluster strongly depend on applications running on said cluster. Different tools require different amount of memory or CPU. Estimated traffic also needs to be considered.

However, absolute minimum for cluster is 2GB of RAM for master node and 1GB of RAM for worker node. At least 1.5 CPU cores for master and 0.7 CPU core for worker.

## Servers
