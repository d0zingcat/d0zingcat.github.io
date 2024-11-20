---
author: ["Me"]
title: 'Hetzner Cloud/Robot in Practice'
date: 2024-11-20T16:22:51+08:00
categories: []
tags: []
draft: true
---


Install Image: https://github.com/hetzneronline/installimage/blob/master/config.sh

Terrafrom provider

There is also one community provider which can simply handle some resources of Robot.

K8s Nodes should turn swap off

# First diasbale swap
sudo swapoff -a

# And then to disable swap on startup in /etc/fstab
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

and ensure the /etc/resolv.conf has no more than two name servers(or you will see a a lot of warnings).

Expose routes to vSwitch

https://docs.hetzner.cloud/whats-new#2023-06-23-networks-expose-routes-to-vswitch

Route between cloud and dedicated server by vSwitch:

https://community.hetzner.com/tutorials/how-to-route-cloudserver-over-private-network-using-pfsense-and-hcnetworks

Use cloud to create k8s cluster:

https://blog.kay.sh/kubernetes-hetzner-cloud-loadbalancer-nginx-ingress-cert-manager/

https://www.felipecruz.es/installing-kubernetes/

https://somestacknet.medium.com/install-a-kubernetes-cluster-with-kubespray-on-hetzner-cloud-servers-1d9ef8e933a6

https://www.cedi.dev/post/prod-ready-kubeone/

https://github.com/dhenkel92/hcloud-kubernetes

https://jmrobles.medium.com/how-to-create-a-kubernetes-cluster-with-rancher-on-hetzner-3b2f7f0c037a

https://www.tonoid.com/kubernetes-on-hetzner-in-5min-monitoring

https://www.linuxtechi.com/install-kubernetes-on-ubuntu-22-04/

https://community.hetzner.com/tutorials/install-kubernetes-cluster

https://vitobotta.com/2023/01/07/kubernetes-on-hetzner-cloud-the-easiest-way/

https://faun.pub/install-a-kubernetes-cluster-on-hetzner-cloud-200d4fb6a423

https://community.hetzner.com/tutorials/setup-your-own-scalable-kubernetes-cluster

https://community.hetzner.com/tutorials/kubernetes-with-claudie

https://community.hetzner.com/tutorials/install-kubernetes-cluster

Start point of installation:

https://docs.hetzner.com/robot/dedicated-server/troubleshooting/hetzner-rescue-system/

Install system:

https://docs.hetzner.com/robot/dedicated-server/operating-systems/installimage/

Configure network(private network by vSwitch)

https://docs.hetzner.com/robot/dedicated-server/network/vswitch/

Connect Cloud and Dedicated server by vSwitch:

https://docs.hetzner.com/cloud/networks/connect-dedi-vswitch/

