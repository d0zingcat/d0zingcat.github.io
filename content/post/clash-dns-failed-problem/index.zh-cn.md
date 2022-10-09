---
title: "Clash DNS failed 问题解决"
description: 
date: 2022-10-08T12:38:24+08:00
image: 
math: 
license: 
hidden: false
comments: true
categories: ['Troubleshooting']
tags: ['clash', 'shadowsocks']
draft: false
---


问题的表现是同样两个的节点（一个是日本的JPOS_1机房，一个是香港的IPLC线路，正常来讲这两个节点在上海/江苏应该都可以访问且延迟是比较低的，而考虑到江苏的网络情况推断日本的节点表现会更加好），日本节点在上海能够正常访问，但回了江苏之后就无法访问了，节点测试 fail ；而使用香港节点虽然可以正常访问，但速度非常缓慢。

本以为是的 ClashX Pro的问题，但发现手机上的QuantumultX两个节点延迟都是50+ms的延迟，而且访问速度很快。删除ClashX Pro重装ClashX并不能解决问题。
进一步排查问题，查看日志可以看到的很关键的一条：

`all DNS requests failed, first error: dial tcp4 1.1.1.1:53`

且观察到IP解析出来都是198.18 ，`Enhanced Mode Config-DNS Mode` 是 `fake-ip` 。考虑到在家里的翻墙方式是R2S的软路由透明代理实现的，且用了Fake-IP的模式，为了尽量避免麻烦和不必要的影响因素，即便fake-ip可以加速代理的速度（节约一次解析DNS的时间），也把ClashX Pro的DNS Mode 改成 Mapping，这样可以保证不在家的情况下使用ClashX 解析出来的结果都是正常的IP而不是软件中的fake-ip，这样尽可能分离开两个fake-ip模式下会有的理解上的不一致（即便可能ClashX Pro的DNS Mode也选fake-ip也不会存在问题，但毕竟没有尝试过）。

当改变了这个设置之后，发现原先配置的节点域名可以正常解析到自己本来的IP而不是fake-ip了，但问题还是存在，访问极慢且提示1.1.1.1的53解析失败了。尝试着把配置中的dns直接关掉也不能解决问题，于是一个一个路径地注释`1.1.1.1` 最后发现是 `fallback` 中的地址链接失败了。
配置为

```plain
dns:
  enable: true
  listen: 0.0.0.0:7874
  ipv6: false
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  nameserver:
  - 114.114.114.114
  - https://1.1.1.1/dns-query
  - 180.168.255.118
  - 116.228.111.18
  - 119.29.29.29
  - https://doh.pub/dns-query
  - https://dns.alidns.com/dns-query
  fallback:
  - tcp://1.1.1.1
  fallback-filter:
    geoip: true
    ipcidr:
    - 240.0.0.0/4
  default-nameserver:
  - 114.114.114.114
  - 119.29.29.29
  - 180.168.255.118
  - 116.228.111.18
  fake-ip-filter:
  - "*.binance.com"
```

根据这个帖子[^1]可以明白clash的解析逻辑是并发请求`nameserver`和`fallback`中的所有的dns服务器，拿到最快的一个结果看是否是的CN的IP，如果不是则返回`fallback`中的解析最快的一个的结果（不是很理解这个逻辑，如果这样的话`fallback`被投毒（不确定可行性）或者连不通不就会导致代理存在问题么？就像我这次碰到的问题一样）。telnet一下可以发现确实在江苏无法正常连通 `1.1.1.1` 这个IP的53端口，至此问题已经确认，那么需要做的就很简单，通过查阅一些别人的配置我在`fallback` 中新增了两个 DNS 服务： 

```plain
dns:
  enable: true
  listen: 0.0.0.0:7874
  ipv6: false
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  nameserver:
    - 114.114.114.114
    - https://1.1.1.1/dns-query
  fallback:
    - tcp://1.1.1.1
    - https://public.dns.iij.jp/dns-query
    - https://doh.pub/dns-query
```

再次尝试之后发现果然在江苏不论是哪个节点都可以正常翻墙，且访问速度非常快，所有网页都能很快打开。

# 参考
[记一次解决 clash all DNS requests failed, context deadline exceeded 问题][2]

[[疑问]dns中fallback的作用][3]

[在 Clash 中 DNS 解析行为和 fallback-filter 的一点理解][4]

[浅谈在代理环境中的 DNS 解析行为][5]

[DNS污染对Clash（for Windows）的影响][6]

[[Feature]希望可以由用户显式设置 Nameserver 和 Fallback DNS 是否通过代理来解析域名 #857][7]

[clash添加doh解析][8]

[DNS设置][9]

[[Feature] 添加根据CNAME指向的FQDN匹配规则][10]

[Home][11]

[DNS][12]


[^1]: [[疑问]有关clash dns 解析中fallback组的意义的疑问][1]

[1]: https://github.com/Dreamacro/clash/issues/968
[2]: https://hellodk.cn/post/870
[3]: https://github.com/Dreamacro/clash/issues/642
[4]: https://cosey.me/p/clash-fallback-filter.html
[5]: https://blog.skk.moe/post/what-happend-to-dns-in-proxy/
[6]: https://github.com/Fndroid/clash_for_windows_pkg/wiki/DNS%E6%B1%A1%E6%9F%93%E5%AF%B9Clash%EF%BC%88for-Windows%EF%BC%89%E7%9A%84%E5%BD%B1%E5%93%8D
[7]: https://github.com/Dreamacro/clash/issues/857
[8]: https://s1oz.github.io/post/clash-tian-jia-doh-jie-xi/
[9]: https://github.com/vernesong/OpenClash/wiki/DNS%E8%AE%BE%E7%BD%AE
[10]: https://github.com/Dreamacro/clash/issues/621#issuecomment-614702773
[11]: https://github.com/Dreamacro/clash/wiki
[12]: https://lancellc.gitbook.io/clash/clash-config-file/dns
