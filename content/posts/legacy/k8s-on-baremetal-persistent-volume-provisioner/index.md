---
author: ["Me"]
title: 'K8s on Baremetal: Persistent Volume Provisioner'
date: 2024-05-07T16:05:24+08:00
categories: []
tags: []
draft: true
---

We run our severs on bare mental and there are a lot of scenarios we have to use a persistent volume or such things where we need to persist some configs. But we don't have a cloud volume to provision and we have to assure the disk IO performance, which means we have to choose one efficient(sometimes maybe not that stable or full-fledged) solution. That's why we choose `local-path-provisoner`.

As the [local-path-provisioner](https://github.com/rancher/local-path-provisioner) has no helm charts, although we can install its manifests through http data sources, it takes a lot of time for terraform to check the status each time you run `terraform apply`, it's a little bit time consuming. So, in this case, we have to make a concession. We choose to install manually.

(Make sure to switch to the current context)

```
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.26/deploy/local-path-storage.yaml
```
