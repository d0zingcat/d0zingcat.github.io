---
author: ["Me"]
title: 'K8s on Baremetal Components'
date: 2024-11-20T16:26:28+08:00
categories: []
tags: []
draft: true
---

Install starrocks

https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/initialize_root_password_howto.md

Install Redis

```
user test on >iiiii +get +get +set +@read +@write +select +ping +eval +multi +exec -@dangerous ~ddd:*
#user default on nopass sanitize-payload ~* &* +@all

#user default on >xxxx sanitize-payload ~* &* +@all
#user replica-user on >yyyy +psync +replconf +ping
#masteruser replica-user
#masterauth a-strong-password
```