---
author: ["Me"]
title: 'Redis Replica Full Sync Failed'
date: 2024-07-22T15:27:35+08:00
categories: []
tags: []
draft: false
---

After a long time running, my redis instance consumes a lot of memory(RDB file takes about 65 GB on Disk). At this time, I want to set a new replication on this standalone instance to keep my data safe, so I start a new replica instance which contains the same ACL permission alike the master instance and add following options to enable Redis replication.

```
    slaveof redis 6379
    masterauth xxxx
```

The replica starts soon from a brand new disk snapshot(a new AOF checkpoint) and begin syncing from the master. But soon it failed. 

Replica logs:

```
1:S 22 Jul 2024 07:31:30.512 * Partial resynchronization not possible (no cached master)1:S 22 Jul 2024 07:40:40.186 * Full resync from master: 579e128b29dcec58b72f3177cba9d614f64c1f48:29975283281:S 22 Jul 2024 07:40:45.947 * MASTER <-> REPLICA sync: receiving streamed RDB from master with EOF to disk
```

Master logs:

```
1:M 22 Jul 2024 07:14:28.043 # Client id=192185 addr=10.233.111.203:56504 laddr=10.233.125.173:6379 fd=14 name= age=246 idle=246 flags=S db=0 sub=0 psub=0 ssub=0 multi=-1 qbuf=0 qbuf-free=0 argv-mem=0 multi-mem=0 rbs=1024 rbp=0 obl=0 oll=4264 omem=90243008 tot-mem=90244936 events=r cmd=psync user=default redir=-1 resp=2 lib-name= lib-ver= scheduled to be closed ASAP for overcoming of output buffer limits.
```

The solution to this should be like below, to add a few options on the master.

```
redis-cli config set client-output-buffer-limit "slave 0 0 0"
redis-cli config set repl-backlog-size 512mb
```

## References:

[redis-client-ouput-buffer-limit](https://gist.github.com/amgorb/c38a77599c1f30e853cc9babf596d634)
