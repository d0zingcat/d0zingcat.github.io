---
author: ["Me"]
title: 'Orange Pi Intro 1'
date: 2024-11-20T16:17:26+08:00
categories: []
tags: []
draft: true
---

安装 tailscale 

curl -fsSL https://tailscale.com/install.sh | sh

\#似乎存在什么问题 在 apt update 时候出错了

Installing Tailscale for debian bookworm, using method apt

\+ mkdir -p --mode=0755 /usr/share/keyrings

\+ curl -fsSL https://pkgs.tailscale.com/stable/debian/bookworm.noarmor.gpg

\+ tee /usr/share/keyrings/tailscale-archive-keyring.gpg

\+ curl -fsSL+  https://pkgs.tailscale.com/stable/debian/bookworm.tailscale-keyring.list

tee /etc/apt/sources.list.d/tailscale.list

\#Tailscale packages for debian bookworm

deb [signed-by=/usr/share/keyrings/tailscale-archive-keyring.gpg] https://pkgs.tailscale.com/stable/debian bookworm main

\+ apt-get update

Hit:1 http://repo.huaweicloud.com/debian bookworm InRelease

Hit:2 http://repo.huaweicloud.com/debian bookworm-updates InRelease

Hit:3 http://repo.huaweicloud.com/debian bookworm-backports InRelease

Get:4 https://mirrors.aliyun.com/docker-ce/linux/debian bookworm InRelease [43.3 kB]

Err:4 https://mirrors.aliyun.com/docker-ce/linux/debian bookworm InRelease

  The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 7EA0A9C3F273FCD8

Get:5 https://pkgs.tailscale.com/stable/debian bookworm InRelease

Get:6 https://pkgs.tailscale.com/stable/debian bookworm/main armhf Packages [9,884 B]

Get:7 https://pkgs.tailscale.com/stable/debian bookworm/main arm64 Packages [9,969 B]

Get:8 https://pkgs.tailscale.com/stable/debian bookworm/main all Packages [354 B]

Reading package lists... Done

W: GPG error: https://mirrors.aliyun.com/docker-ce/linux/debian bookworm InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 7EA0A9C3F273FCD8

E: The repository 'https://mirrors.aliyun.com/docker-ce/linux/debian bookworm InRelease' is not signed.

N: Updating from such a repository can't be done securely, and is therefore disabled by default.

N: See apt-secure(8) manpage for repository creation and user configuration details.

手动安装一下

apt install tailscale -y

systemctl enable tailscaled

systemctl start tailscale

tailscale up

\#login...

tailscale netcheck

给 eMMC 刷入了系统，打开后插入了 TF 卡（可以 hot plugging），然后读取挂载。但是当关机重启之后系统无法正常进入，CPU 灯不闪烁，怀疑可能是在插入 TF 卡的时候会优先从 TF 卡中加载系统（可能和出厂的时候使用的 TF 卡测试，所以使用 eMMC 的时候需要提前 clear SPIFlash 是一样的意思）。因此需要重新刷机。

可能是因为我的 windows 做了 hard disk check 或者系统文件修复等等，也可能是 orangepi 刷机太多次的缘故（我觉得这个概率稍小），我的 pc 无法正常读取 maskrom （之前是一开机立马就读取到，现在要过很久，且一下载 boot 就会断开，导致我无法清除 SPIFlash），也无法正常给 eMMC 刷入系统，因此只能选择使用 baleEtcher 给 TF 卡刷入系统，然后格式化 eMMC 作为数据存储。

刷入了 ubuntu jammy server 版
