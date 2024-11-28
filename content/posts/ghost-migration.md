```yaml
author: ["Me"]
title: 'Ghost Migration'
date: 2024-11-27T08:10:35+08:00
tags: ["Ghost", "Migration", "K8s", "Docker"]
categories: ["DevOps"]
draft: false
```

# Migration from Ghost(Docker) to Ghost(K8s)

Recently I am prepparing for migrating my blog from Ghost(Docker on Baremetal) to Ghost(on K8s).

## Changes 

| Previous | Previous  |  Now |
|---|---|---|
| Host| Docker | K8s |
| Database | Mariadb | Mysql |
| S3 | Cloudflare R2 | Cloudflare R2 |
| Route | Caddy | Ingress |

## Steps

### Boot up the new Ghost

I am using the official docker image[^2] to boot up the new Ghost. I am quite sure the official image is the most suitable one for me to run instead of bitnami one(it has way too many self defined logic processes in it). I don't want to use the bitnami helm charts either(I use helm to install all the things on my K8s Cluser).

[^2]: [Ghost](https://hub.docker.com/_/ghost)

The other things do not matter, as we can focus on the `envs`(Don't forget to persistent the data dir or the adapter will get lost when you next reboot the pod).

```
env:
- name: url
  value: "https://example.com"
- name: debug
  value: "ghost:*,ghost-config"
- name: database__client
  value: "mysql"
- name: database__connection__host
  value: "mysql-host"
- name: database__connection__user
  value: "ghost-user"
- name: database__connection__password
  value: "your-password"
- name: database__connection__database
  value: "ghost-db"
- name: mail__transport
  value: "SMTP"
- name: mail__options__service
  value: "your-mail-service"
- name: mail__options__auth__user
  value: "your-email@example.com"
- name: mail__options__auth__pass
  value: "your-mail-password"
- name: mail__from
  value: "'Your Name' <noreply@example.com>"
# - name: adapters__storage__active
#   value: "s3-ghost"
# - name: adapters__storage__s3-ghost__accessKeyId
#   value: "your-access-key-id"
# - name: adapters__storage__s3-ghost__secretAccessKey
#   value: "your-secret-access-key"
# - name: adapters__storage__s3-ghost__bucketName
#   value: "your-bucket-name"
# - name: adapters__storage__s3-ghost__region
#   value: "auto"
# - name: adapters__storage__s3-ghost__baseDir
#   value: "ghost/"
# - name: adapters__storage__s3-ghost__assetsBaseUrl
#   value: "//your-assets-url.com"
# - name: adapters__storage__s3-ghost__endpoint
#   value: "https://your-endpoint-url.com"
# - name: adapters__storage__s3-ghost__ghostDirectory
#   value: "../../../../current"
```

As you can see, I have commented the `adapters__*` as I have not installed the s3-ghost adapter yet.
BTW, the ghost-s3 adapter requires the following configs:

```
 "storage": {
        "active": "s3-ghost",
        "s3-ghost": {
            "accessKeyId": "<AWS key>",
            "secretAccessKey": "<AWS Secret key>",
            "bucketName": "<bucket name>",
            "region": "<region>",
            "assetsBaseUrl": "<CDN url or s3 url>",
            "ghostDirectory": "../../../.."
        }
```

But I config all the configs in `env`, so I have to follow the Ghost [Config Env Conversion](https://ghost.org/docs/config/#configuration-options).
To be simple, for nested config options, separate with two underscores. 

### Backup all the data

I have to migrate following things: 
- Posts
- Images(as I am using s3 to host the images, so I don't need to backup the images folder)
- [Rotues](https://ghost.org/docs/themes/routing/) and Redirects
- Members
- Themes(I am using the default theme so I don't have to)

[This post](https://ghost.org/docs/migration/ghost/) covers the things above, except for [members](https://forum.ghost.org/t/migrate-ghost-to-another-host/44084).

### Inside the pod install the s3 adapter

I am using [s3-ghost](https://github.com/tranvansang/s3-ghost) to host the Object Storage things, as I failed to install the official one(on Ghost Docs[^1]).

[^1]: [Amazon S3 + Ghost](https://ghost.org/integrations/amazon-s3/)

Enter the pod the default location is `/var/lib/ghost/` and run the following command to install.

```
rm -rf content/adapters/storage/s3-ghost
mkdir -p content/adapters/storage/s3-ghost
cd content/adapters/storage/s3-ghost
yarn init -y
yarn add s3-ghost
cp node_modules/s3-ghost/index.js .
rm -rf node_modules/s3-ghost package.json yarn.lock
```

### Recover the adapter things

Remove the `adapter` envs and restart the pod, then everything should be ok.

## Attention

One attention point is that if specify the adapater related settings, ghost will automatically looking for the adapater code and run, if at this point you haven't installed the adapter code, ghost will fail to start.

Another attention point is that if you want to use the s3 adapter, you need to make sure that the `url` keep the same with previous one, otherwise the images will fail to load(But I have no idea how to fix this yet), as the image `url` raw data is like `<img src=\"__GHOST_URL__/content/images/2024/08/image-2.png`.

