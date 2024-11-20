---
author: ["Me"]
title: 'Helm stucking at "Uninstalling" Status'
date: 2024-04-06T16:18:20+08:00
categories: ["DevOps"]
tags: ["Helm", "Kubernetes", "Troubleshooting"]
draft: false
---

Sometimes if we do not pay attention to the helm release uninstallation order, like we removed one release which includes CRDs and then we try to remove a customized resource which does not have a reference or finalization, that means this resource will cause resource recycle stuck, in other words, the helm release stuck at "uninstalling" status.

There is a quick fix which can easily sort it out.

```
helm plugin install https://github.com/helm/helm-mapkubeapis

helm mapkubeapis RELEASE_NAME
helm del RELEASE_NAME
```

If this did not help, then delete secret for each version

```yaml
helm hist releasename
kubectl get secrets
k delete secrets sh.helm.release.v1.name.VERSION-N
```

## References

https://github.com/helm/helm/issues/11513#issuecomment-1461323751

https://github.com/helm/helm-mapkubeapis

https://medium.com/@calvineotieno010/no-i-cannot-delete-it-when-helm-refuses-to-delete-a-release-ac9c64919e2b
