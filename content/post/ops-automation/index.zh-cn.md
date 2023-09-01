---
title: "Ops Automation Intro"
description: 
date: 2023-08-30T22:51:41+08:00
image: 
math: 
license: 
hidden: false
comments: true
categories: ['Tutorial']
tags: ['automation', 'ops', 'terrafom', 'ansible', 'cloud-init']
draft: false
---

> 之前[^1]有介绍过我一般会在一台新的代理服务器上做的一般性的操作，这边对该文做了一定的拓展和补充。

事实上，如果我们存在多台服务器的时候，一台一台配置必然会耗费大量的精力，也非常容易出现配置不一致的问题，最后往往会引发一些不必要的麻烦，一般问题查到最后总是一个非常低级的错误。那么有没有办法避免这样的问题呢？答案自然是有的，那就是自动化。

考虑到自动化我一般可能会选择以 Terraform 为代表的云服务管理工具，如果是普通的大众的云服务器，基本上都提供了 API（以及 terraform 的 provider）。但自己折腾的过程中往往还有这两种情况：

1. 奇奇怪怪的小服务器厂商，人家根本没有对应的 terraform 支持
2. 独立主机（可能是 baremetal，也可能是自己局域网里面的主机，一般我会选择使用 Proxmox VE 来管理，但也可能是别种虚拟软件）

因此，在这个层面上来讲，terraform 并不是万能的。我开始思考有没有别的同类型的工具，通过简单的调研发现还有两种：Cloud-init 和 Ansible。

前者我在使用 PVE 的时候就已经发现了，但 PVE 原生对其支持很有限（只能用来配置 network相关），并没有很大灵活性；而我是后续在使用 Digital Ocean 或是 Vultr 这类云厂商的时候发现创建主机可以指定一个 user-data ，按照 cloud-init 的语法编写对应的配置就可以非常简单和高效地完成初始化。

后者我在很早的时候就便听说了，且现在的公司使用的是 Hetzner Bare Metal 主机运维就是通过这个工具来完成来的，这次我选择使用这个工具来完成我 AWS Lightsail 和 Oracle Cloud 上各一台小主机的自动化运维。事实上，这次我本想要通过 cloud-init + terraform 的方式来运维，主要考虑到我的其他云设施是通过 terraform 来管理的（例如 Cloudflare 的域名），而 clout-init 在其他云厂商被支持的比较好，也是一个通用且强大的解决方案，一定程度上是“一剑破万法的”，但实际调研的过程中发现 cloud-init 没有基础设施的支持（例如一个局域网的 HTTP 服务用于提供 metadata 的下载）的话要使用本地的配置文件这件事情就非常复杂，而 terraform 的话因为已经是开好的两个 VPS 而无法直接定义成对应的资源（也更别提自动化的配置了），也无法和 cloud-init 结合起来用。上述只是我在查阅资料过程中得到的浅薄的结论，如有错误，恳请指正。

回到 ansible 本身，首先是安装，mac 用户直接 brew。
然后建立一个 ansible 的目录，结构如下：
```
➔ tree
.
├── docker-compose.yml
├── inventory.yaml
└── playbook.yaml

1 directory, 3 files
```

`docker-compose.yml` 使用 docker 启动一个 gost 的 instance 用于翻墙，内容如下：

```yaml
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
      - ss+ssu://chacha20-ietf-poly1305:{password}@:3388
      ports:
      - 3388:3388/tcp
      - 3388:3388/udp
      restart: unless-stopped
        
networks:
  my_network:
```

`inventory.yaml` 中定义了我要运维的两台主机：
```yaml
virtualmachines:
  vars:
    ssh_port: 1984
  hosts:
    oracle_sanjose:
      ansible_host: {hosta_ip}
      ansible_user: ubuntu
    aws_tokyo:
      ansible_host: {hostb_ip}
      ansible_user: admin
```

`playbook.yaml` 中定义了具体的操作：
```yaml
---
- name: Init
  hosts: virtualmachines
  vars:
    user: d0zingcat
  tasks:
    - name: Install deps
      become: true
      apt:
        name:
          - htop
          - acl
          - nload
          - tree
          - vim
          - iptables
          - iptables-persistent
        state: latest
    - name: Upgrade
      become: true
      apt:
        upgrade: dist
    - name: Add User {{ user }}
      become: true
      user: 
        name: "{{ user }}"
        shell: /bin/bash
        generate_ssh_key: yes
        groups: "{{ user }},sudo"
    - name: "Add authorized keys"
      become: true
      authorized_key:
        user: "{{ user }}"
        key: "{{ lookup('onepassword', 'Personal', field='public key') }}"
    - name: Exec sh script on Remote Node
      become: true
      shell:
        cmd: |
          iptables -F
          iptables -X
          iptables -P INPUT ACCEPT
          iptables -P FORWARD ACCEPT
          iptables -P OUTPUT ACCEPT
          netfilter-persistent save
          cat <<EOF | tee /etc/sysctl.d/common.conf
          net.core.default_qdisc=fq
          net.ipv4.tcp_congestion_control=bbr
          net.ipv4.ip_forward=1
          net.ipv6.conf.all.forwarding=1
          EOF
          sysctl -p
          cat <<EOF | tee /etc/sudoers.d/100-ansible-user
          "{{ user }}" ALL=(ALL) NOPASSWD:ALL
          EOF
          sed -i '/#Port /s_^#Port 22_Port 1984_' /etc/ssh/sshd_config
          sed -i '/^#PermitRootLogin prohibit-password/s_#__' /etc/ssh/sshd_config
          hostnamectl set-hostname myworker
          # systemctl restart sshd
          curl -fsSL https://tailscale.com/install.sh | bash
          curl -fsSL https://get.docker.com | bash
          systemctl enable docker
          usermod -aG docker {{ user }}
      register: result
    - name: Copy docker compose
      become: true
      become_user: "{{ user }}"
      copy:
        src: ./docker-compose.yml
        dest: /home/{{ user }}/docker-compose.yml
        backup: false
    - name: Show result
      debug:
        msg: "{{ result.stdout }}"
```
简单解释一下👆的配置的作用：
- 安装依赖，并升级当前所有的软件包，`become: true` 意味着提权到 sudo（inventory 中配置的账户应该具备 sudo 免密的权限）
- 添加一个用户，用户名在 `vars` 中定义了，替换语法即 "{{ user }}"
- 对用户添加 authorized_keys 用于 ssh 免密登陆，`"{{ lookup('onepassword', 'Personal', field='public key') }}"` 用于查询 `1Password` 中的 Personal 项目，字段是 `public key`
  - 如果是本地文件的话可以换成：`"{{ lookup('file', 'files/'+ 'xxx' + '.key.pub') }}"`
- 执行提前写好的 shell 脚本
  - 脚本的主要作用如下
    - 打开 iptables 防火墙并持久化
    - 更换 TCP 拥堵算法为 BBR，打开流量转发
    - 给👆添加的用户添加上 sudo 免密权限
    - 修改 sshd 配置，更换非标端口并禁止 root 用户密码登陆
      - 可以发现我并没有重启 sshd服务，一个小技巧是可以先连接上服务器之后再手动重启，然后再开一个终端连接，如果能连上则意味着修改成功；如果不能则表示存在一些问题，这个时候可以用已经连上的会话进行回滚，以免服务器失联（在忘了修改防火墙的时候这种修改策略非常有效）
    - 修改 hostname
    - 安装并自启动 docker，添加当前用户到 docker 的组，这样不用每次敲 sudo
  - 在编写自己的脚本的时候可以明确自己的实现方式，尽量使用声明式而非过程式。举个🌰，在添加 sysctl 的时候可以使用 `echo 'xxx=yyy' >> /etc/sysctl.conf` 进行操作，但这样会有一个问题，当多次执行 ansible 的时候就会添加很多配置，换句话说，这种操作并不是幂等的。而如果脚本中再去处理之前有没有添加过这样的配置这件事情就会变得比较复杂，因此可以换个思路，每次都在 `/etc/sysctl.d/` 下新建一个配置文件，重新写入期望的配置，这样不论执行多少次配置文件都是预期内的，也是干净的。建议编写自己的脚本的时候都带着这样的思路去实现，对于维护整体的稳定性和简单性有比较大的帮助。
- copy 本地的 docker compose 文件到服务器
- 打印执行结果

`ansible-playbook -i inventory.yaml playbook.yaml` 执行一下等待完成即可。

经过上面的操作之后事情就比较简单了，剩下我需要手动做的事情还有：
- 添加本地的 ssh config
- 连接到对应的服务器
  - 重启 sshd 使配置文件生效
  - 启动 docker compose，也就是 `docker compose up -d`
  - 登陆 tailscale `sudo tailscale up --accept-routes`
  - 添加 /etc/hosts 保证 hostname 有解析

之所以这样选择是这几步操作有的可能导致问题，是高危操作，我希望看到错误日志或者需要干预；有的是无法自动化，得手动处理；有的就是手动操作很简单但写脚本比较复杂。

[^1]: [如何配置一台新的翻墙服务器](https://blog.d0zingcat.dev/p/how-to-setup-a-new-vpn/)