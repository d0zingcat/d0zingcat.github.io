---
author: ["Me"]
title: 'Helm Install MySQL Cluster'
date: 2024-11-29T20:16:59+08:00
categories: ["Database", "Kubernetes"]
tags: ["mysql", "helm", "k8s"]
draft: false
---

## Comparison

I have to disclaim that due to my limitation of knowledge, the following is just my personal opinion(according to my experience). 

So there may be some errors or omissions. If you have any suggestions, please let me know.

| Feature | [MySQL Operator](https://github.com/mysql/mysql-operator) | [Bitnami MySQL](https://github.com/bitnami/charts/tree/main/bitnami/mysql) | [Percona XtraDB Cluster](https://github.com/percona/percona-xtradb-cluster-operator) |
|---------|----------------|---------------|------------------------|
| High Availability | Yes | Yes | Yes |
| Automatic Failover | Yes | No | Yes |
| Backup/Restore | Yes | No | Yes |
| Scaling | Yes | Limited | Yes |
| Custom Configurations | Yes | Limited | Yes |
| Multi-Master Replication | Yes | No | Yes |
| Ease of Setup | Moderate | Simple | Moderate |
| Community Support | Weak | Strong | Strong |
| Cloud Native | Yes | No | Yes |

## Installation

### Bitnami MySQL

As to my concern, the bitnami one does not have a failover solution, so if the mater is down, it will be down for sure. As a matter of fact, it's quite difficult to fix it. 

Let me take one example, if the master host's disk corrupted, the master will be down. But, how do you plan to recover it? The disk is corrupted, and there's no way to fix it. 

You have never backed up your data, so you have only one choice, which is to replace the master host. However, the data on the replication host is not usable, you can't just start a new master and copy the data from the old one(or the replicated one).

You have to export the data, users and permissions from the replication and then import it to the new master. What if the database is quite big like 2TB? How long will it take? It's not a matter of minutes, it's a matter of hours. So we can never accept the solution. 

Then how about promote the slave to master? It's not a good idea when you deployed the cluster on K8s, as the service points to the desired instances, like if you promote the slave to master, you have to change all your apps' DSN to point to the replicated one, e.g. `mysql-secondary` even tough it's not a slave anymore. Also, you have lost the choice to recover the master(or you have to boot up the master with all new data, manully join to the new master and make it a slave, meanwhile the old `mysql-primary` service points to this new salve, it's a mess). You can also modify the selector of the services to point to the correct pod, but then you've lost the future chances to upgrade the helm chart, which is also a bad idea.

Although bitnami MySQL doesn't have a failover solution, it has a nice feature: it can scale up to 3 replicas, so it's a good choice for HA. So I will also introduce how to install it.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install my-mysql bitnami/mysql --values values.yaml
```

I can show you my values file:

```yaml
nameOverride: mysql
architecture: replication
auth:
  existingSecret: mysql-password
primary:
  extraFlags: "--max-connections=302"
  nodeSelector:
    abc: "true"
  persistence:
    size: 100Gi
    storageClass: local-path
  resources:
    requests:
      memory: "4Gi"
      cpu: "2"
  service:
    type: NodePort
    nodePorts:
      mysql: '33001'
secondary:
  extraFlags: "--max-connections=302"
  replicaCount: 1
  persistence:
    size: 100Gi
    storageClass: local-path
  resources:
    requests:
      memory: "2Gi"
      cpu: "1"
metrics:
  enabled: true
  serviceMonitor:
    enabled: true
    relabelings:
    - sourceLabels: [__meta_kubernetes_pod_name]
      targetLabel: "instance"
  resources:
    limits:
      cpu: 1.5
      ephemeral-storage: 1Gi
      memory: 512Mi
    requests:
      cpu: 100m
      ephemeral-storage: 50Mi
      memory: 128Mi
```
I've set the cluster:

- Pre craete a secret to store the password
- Use `local-path` storage class from [Rancher](https://github.com/rancher/local-path-provisioner) to store the data
- Increase the max connections to 302
- Enable metrics(as I want to use grafana panel to monitor it)
- Open a NodePort to access it
- Set resources of cpu and memory

### MySQL Operator

This solution looks quite fancy to me comparing to the bitnami one. It's a good choice if you want to use the MySQL Operator to manage your MySQL cluster, as it:

- Has a failover solution
- Can scale up to replicas(but be an odd number, based on Paxos selection)
- Can use multi-master replication
- Has backup solution
- From MySQL Official team


Install the operator: 

```bash
helm repo add mysql-operator https://mysql.github.io/mysql-operator/
helm repo update
helm install mysql-operator mysql-operator/mysql-operator
```

Then I can install the innodbcluster:

```bash
helm install mysql-operator mysql-operator/mysql-innodbcluster --values values.yaml
```

Helm values file(I've not enabled a S3 backup as I don't have one yet):

```yaml
credentials:
  root:
    user: root
    password: '{my_password}'
    host: '%'
serverInstances: 1
routerInstances: 1
tls:
  useSelfSigned: true
datadirVolumeClaimTemplate:
  storageClassName: hcloud-volumes
  accessModes: ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

If you want to pre create the mysql password, you can also use this way:

```bash
$> kubectl create secret generic mypwds \
        --from-literal=rootUser=root \
        --from-literal=rootHost=% \
        --from-literal=rootPassword="sakila"
```

```bash
cat <<<EOF | kubectl apply -f - 
apiVersion: mysql.oracle.com/v2
kind: InnoDBCluster
metadata:
  name: mycluster
spec:
  secretName: mypwds
  tlsUseSelfSigned: true
  instances: 3
  router:
    instances: 1
EOF
```

Sadly, the helm chart is still quite simple so you can't use the helm chart to install if you want to refer to an existing secret to store the password.

You can easily find the fallbacks of this choice:

- No community support(They don't accept any PRs and they don't event have an Issues page, they only ask you to open a bug on their bug tracker)
- The operator is based on Python, which means not easy to extend

### Percona XtraDB Cluster

It's quite facinating to me as it says: 

> Percona XtraDB Cluster (PXC) is a 100% open source, enterprise-grade, highly available clustering solution for MySQL multi-master setups based on Galera.PXC helps enterprises minimize unexpected downtime and data loss, reduce costs, and improve performance and scalability of your database environments supporting your critical business applications in the most demanding public, private, and hybrid cloud environments.

And from the Percona Server for MySQL it also mentions PXC:

> As of today, we recommend using Percona Operator for MySQL based on Percona XtraDB Cluster, which is production-ready and contains everything you need to quickly and consistently deploy and scale MySQL clusters in a Kubernetes-based environment, on-premises or in the cloud.

Which means it's a good choice if you want to use the Percona XtraDB Cluster to manage your MySQL cluster.

I appreciate the highlights of the Percona XtraDB Cluster:

- Production-ready
- Open to community
- Multi-master replication, each node can accept any request
- Has backup solution
- Fully compatible with MySQL Server Community Edition 8.0 

However, it also has some downsides:

- Whenever provisioning a new node, it will copy the full data set
- Limitations 
  - LOCK/UNLOCK TABLES clause is not supported
  - Each table should have a primary key or same query on different replica may cause cardinary disorder
  - Other... Not quite important to me as I don't use it


