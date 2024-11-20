---
author: ["Me"]
title: 'K8s and Rbac'
date: 2024-11-20T15:30:54+08:00
categories: []
tags: []
draft: true
---

```
_list=($(kubectl get --raw / |grep "^    \"/api"|sed 's/[",]//g')); for _api in ${_list[@]}; do _aruyo=$(kubectl get --raw ${_api} | jq .resources); if [ "x${_aruyo}" != "xnull" ]; then echo; echo "===${_api}==="; kubectl get --raw ${_api} | jq -r ".resources[].name"; fi; done
```
