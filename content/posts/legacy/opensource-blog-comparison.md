---
author: ["Me"]
title: 'Opensource Blog Comparison'
date: 2024-11-20T16:09:57+08:00
categories: []
tags: []
draft: true
---

> 在 [NodeSeek](https://www.nodeseek.com/post-79831-1?ref=hammerplease.uk) 上面也有一个讨论，有兴趣可以参考或者参与讨论。

## 调研

### 诉求

- must 开源
- must 有管理界面 可以随开随写
- must 支持 markdown 写作
- better 支持插件
- better 把图片存在 s3
- better docker 部署

### 评价

- [wordpress](https://wordpress.com/?ref=hammerplease.uk) 都满足 但是笨重 复杂 真的难顶
- [typecho](https://typecho.org/?ref=hammerplease.uk) 复活没多久 插件不多 社区不活跃 php不熟悉写插件难顶
- [ghost](https://ghost.org/?ref=hammerplease.uk) 插件系统简直了。。docker 里面装插件搞了一天了没搞定 社区也不活跃 几百年前的问题没人回答

### 网友推荐

- wordpress
- [halo](https://www.halo.run/?ref=hammerplease.uk)
- [NotionNext](https://www.tangly1024.com/?ref=hammerplease.uk)
  - 还发现了两个相似的，一看就非常 fancy
  - [nextjs-notion-starter-kit](https://github.com/transitive-bullshit/nextjs-notion-starter-kit?ref=hammerplease.uk)
  - [notion-blog-nextjs](https://github.com/samuelkraft/notion-blog-nextjs?ref=hammerplease.uk)
- ...
- 静态博客因为切换设备的时候无法连续操作，以及必须得在电脑上操作（可能结合CI也可以做到手机上写，但觉得麻烦）遂放弃
  - 可能可以考虑直接配合 github 网页来进行写作 可以多设备
- 还有一些小众的就不写了，可以爬楼

## 决策

关于调研中我提到的 wordpress、typecho、ghost 我都在机器上面安装体验过了，且简单的配置了一下。

wordpress
插件

s3

主题简洁优美

seo

sitemap

rss

轻量 响应迅速

markdown 写作

之所以执着于 s3 是可能 mvc 惯了，总想要解耦，而且服务器本身硬盘我只有50g，不可能无限扩容。同时备份成本也会因为静态文件的增多而变得缓慢以及存储成本提升（虽然也可以做增量备份但又引入了复杂性）。于是就把这个作为一个突破口开始尝试在3个选择之间对比。且我现在只用cloudflare的r2作为我的对象存储。

wordpress 是最轻松的，安装一个mediacloud 就解决了问题，配置也很简单。
typecho 直接没有方案，能找到的都是一些定制化的方案（供应商做的，例如腾讯云cos），没有一个兼容s3协议普适的插件来给我配置 cloudflare的 r2。
ghost按理说很简单，实现一个storage adapter既可，官方也很早就适配了，但多数插件都是几年前更新的，还是1.0的时候的实现（现在5了），落后aws-sdk有一段时间了。且官方文档也很久没有更新相关的细节，落后ghost主版本号也很久了。个别看起来比较新的插件也不知道怎么配置，折腾了两天才找到一个可以用的且配置好了。

## Ghost 相关

https://ghost.org/integrations/custom-integrations/
