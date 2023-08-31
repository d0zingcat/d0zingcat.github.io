---
title: "如何配置一台新的翻墙服务器"
slug: how-to-setup-a-new-vpn
description: 
date: 2023-05-08T15:02:57+08:00
image: 
math: 
license: 
hidden: false
comments: true
categories: ['Journal']
tags: ['vpn','ss']
draft: false
--- 

对我而言，配置一台新的用于翻墙的服务器大概步骤如下：
- 购买&配置
- 安装必备软件
    - `apt update && apt install -y vim nload htop sudo nodejs npm`
    - `npm i -g pm2`
- 配置新账户
    - adduser 交互式添加
    - `usermod -aG sudo {username}`
    - 免密登陆 `vim /etc/sudoers` Add `{username} ALL=(ALL) NOPASSWD: ALL`
    - 配置密钥登陆 `ssh-keygen`
        - 添加 `.ssh/authorized_keys`
        - 设置权限 `chmod 600 .ssh/authorized_keys`
        - 贴入自己的公钥
    - 重新登陆
- 设置ssh config `vim /etc/ssh/sshd_config`
    - 修改登录port为高位端口 `Port {port}`
    - 关闭root的password login `PermitRootLogin prohibit-password`
    - 重启`systemctl restart sshd`
    - 重启之前需要先检查服务器是否启用了 firewalld 或者 iptables ，避免因为防火墙原因无法链接，也需要检查一下云服务商是否也有类似于防火墙的设置，有的话放行端口
- 配置bbr
    - `vim /etc/sysctl.conf`
        - 添加 `net.core.default_qdisc=fq`
        - `net.ipv4.tcp_congestion_control=bbr`
        - `net.ipv4.ip_forward=1`
        - `net.ipv6.conf.all.forwarding=1`
    - `sysctl -p` 生效
- 这两步不一定必须，现在我更喜欢用 docker 来管理我所有的服务
    - 配置gost
        - `wget https://github.com/go-gost/gost/releases/download/v3.0.0-rc7/gost_3.0.0-rc7_linux_amd64.tar.gz`
            - `tar zxvf xxx && mv gost /usr/local/bin`
        - wiki [`https://gost.run/tutorials/protocols/ss/`](https://gost.run/tutorials/protocols/ss/)
        - github `https://github.com/go-gost/gost`
    - 配置pm2
        - `mkdir pm2 && cd pm2 && pm2 init`
        - 贴入关键配置 `gost -L ss+ssu://chacha20-ietf-poly1305:{password}@:8338`
        - `pm2 start`
        - `pm2 startup`
        - `pm2 save`
        - 配置log rotate
            
            ```bash
            pm2 install pm2-logrotate
            pm2 set pm2-logrotate:max_size 50M
            pm2 set pm2-logrotate:retain 10
            pm2 set pm2-logrotate:compress true
            ```
        - shortcuts `pm2 ls / pm2 log/ pm2 monit`
- 安装 docker 
  - `curl -fsSL https://get.docker.com | bash`
  - `systemctl start docker && systemctl enable docker && usermod -aG docker $USER && exit`
- docker 中启动 gost
  ```
  ---
  version: "3.8"
  services:
    gost:
        container_name: gost
        image: gogost/gost
        networks:
        - my_network
        command:
        - -L
        - ss+ssu://chacha20-ietf-poly1305:xxx@:3388
        ports:
        - 3388:3388/tcp
        - 3388:3388/udp
        restart: unless-stopped
  ```
- 安装tailscale
    - `curl -fsSL https://tailscale.com/install.sh | sh`
- 更新openclash
    - `https://github.com/vernesong/OpenClash/releases`
- 配置代理链
    - [https://xiaorun.me/f672d5a3f4484c7ca098d99f8c98e453](https://xiaorun.me/f672d5a3f4484c7ca098d99f8c98e453)
    - [https://t.me/QuanX_API/133](https://t.me/QuanX_API/133)
- 流媒体测速
    - `bash <(curl -L -s check.unlock.media)`
