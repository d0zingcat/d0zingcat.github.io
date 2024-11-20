---
author: ["Me"]
title: 'ArgoCD intermittent updates to manifest fail'
date: 2024-04-30T16:30:17+08:00
categories: ["DevOps"]
tags: ["ArgoCD", "Kubernetes", "CI/CD", "Troubleshooting"]
draft: false
---

Under our scenario we put Github Actions to use argocd cli to update our app's manifest, which is like below

```
      - name: Update ArgoCD Image
        uses: clowdhaus/argo-cd-action/@main
        if: ${{ inputs.argocd_app_name != '' }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          version: 2.6.7
          command: app set ${{ inputs.argocd_app_name }}
          options: |
            --server ${{ vars.ARGOCD_URL }}
            --kustomize-image ${{ fromJSON(steps.meta.outputs.json).tags[0] }}
            --auth-token ${{ secrets.MEEX_ARGOCD_TOKEN }}
```

In this case, the cli will update argocd app manifest of `source.kustomize.images` according to your image.

```
project: default
source:
  repoURL: 'https://github.com/myteam/gitops.git'
  path: apps/test/overlays/test
  targetRevision: main
  kustomize:
    images:
      - 'ghcr.io/myteam/helloworld:sha-f205e29'
destination:
  server: 'https://kubernetes.default.svc'
  namespace: default
syncPolicy:
  syncOptions:
    - CreateNamespace=false
```

Most of our practices works well but sometimes the update would not succeed at all. If we take a deep look into the process, we will find the logs.

```
time="2024-04-29T07:30:16Z" level=info msg="finished unary call with code OK" grpc.code=OK grpc.method=Get grpc.service=cluster.SettingsService grpc.start_time="2024-04-29T07:30:16Z" grpc.time_ms=0.92 span.kind=server system=grpc
time="2024-04-29T07:30:30Z" level=info msg="received unary call /application.ApplicationService/Get" grpc.method=Get grpc.request.content="name:\"helloworld\" appNamespace:\"argocd\" " grpc.service=application.ApplicationService grpc.start_time="2024-04-29T07:30:30Z" span.kind=server system=grpc
time="2024-04-29T07:30:30Z" level=info msg="finished unary call with code OK" grpc.code=OK grpc.method=Get grpc.service=application.ApplicationService grpc.start_time="2024-04-29T07:30:30Z" grpc.time_ms=4.935 span.kind=server system=grpc
time="2024-04-29T07:30:31Z" level=info msg="received unary call /application.ApplicationService/Update" grpc.method=Update grpc.request.content="%!v(PANIC=String method: reflect.Value.Bytes of non-byte slice)" grpc.service=application.ApplicationService grpc.start_time="2024-04-29T07:30:31Z" span.kind=server system=grpc
time="2024-04-29T07:30:32Z" level=info msg="admin updated application spec" application=helloworld dest-namespace=default dest-server="https://kubernetes.default.svc" reason=ResourceUpdated type=Normal user=admin
time="2024-04-29T07:30:32Z" level=info msg="finished unary call with code OK" grpc.code=OK grpc.method=Update grpc.service=application.ApplicationService grpc.start_time="2024-04-29T07:30:31Z" grpc.time_ms=1360.807 span.kind=server system=grpc
time="2024-04-29T07:30:33Z" level=info msg="received unary call /application.ApplicationService/GetApplicationSyncWindows" grpc.method=GetApplicationSyncWindows grpc.request.content="name:\"helloworld\" appNamespace:\"argocd\" " grpc.service=application.ApplicationService grpc.start_time="2024-04-29T07:30:33Z" span.kind=server system=grpc
time="2024-04-29T07:30:33Z" level=info msg="received unary call /repository.RepositoryService/GetAppDetails" grpc.method=GetAppDetails grpc.request.content="%!v(PANIC=String method: reflect.Value.Bytes of non-byte slice)" grpc.service=repository.RepositoryService grpc.start_time="2024-04-29T07:30:33Z" span.kind=server system=grpc
```

Sometimes the `UpdateSpec` method also fails.

```
time="2024-04-29T07:17:02Z" level=info msg="received unary call /application.ApplicationService/UpdateSpec" grpc.method=UpdateSpec grpc.request.content="%!v(PANIC=String method: reflect.Value.Bytes of non-byte slice)" grpc.service=application.ApplicationService grpc.start_time="2024-04-29T07:17:02Z" span.kind=server system=grpc
time="2024-04-29T07:17:04Z" level=info msg="finished unary call with code OK" grpc.code=OK grpc.method=UpdateSpec grpc.service=application.ApplicationService grpc.start_time="2024-04-29T07:17:02Z" grpc.time_ms=1269.342 span.kind=server system=grpc
```

I searched for the error '%!v(PANIC=String method: reflect.Value.Bytes of non-byte slice)' and found someone who use argocd 2.7.1 also faced this problem(mine is 2.6.7). Currently I have no idea of how to deal with it. 

Although the occurrence frequency is not high, occasional trust issues may arise, and there is no guarantee every release will be successful. If one has to check whether each release is successful, the mental burden will increase substantially, and CD will become meaningless.
