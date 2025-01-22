---
author: ["Me"]
title: 'OAuth2 Proxy Practice'
date: 2025-01-22T20:12:59+08:00
categories: ["kubernetes", "oauth2-proxy"]
tags: ["oauth2-proxy", "kubernetes", "prometheus", "alertmanager"]
draft: false
---

# Introduction
[OAuth2-proxy](https://github.com/oauth2-proxy/oauth2-proxy) is a reverse proxy which provides authentication with Google, Azure, OpenID Connect and many more identity providers.

For me, we have prometheus and alertmanager on our K8s cluster. We can use oauth2-proxy to proxy the requests to prometheus and alertmanager. That's quite essential for a robust and secure monitoring system.

However, the above two services do not come with an out-of-box authentication module, which means if we expose them to the internet, everyone can access them. So we have to use oauth2-proxy to expose them to the internet. If you found that the official docs confusing and lack details, then you may find the right guidance for you.

# Prerequisites 

- K8s Cluster
- Github Account

Although it supports a lot of identity providers(with Google by default), since I am using Github Oauth App, I'm goint to show you how to use this as the provider and how to setup it.

Let's take Hashicorp Vault as an example. The installation will be ommitted as it's quite direct and simple. For me, I use [Helm](https://github.com/hashicorp/vault-helm/blob/main/values.yaml) to install this component.

Now, we can assume the vault has been installed already, then you may have one question: How to access the management UI? Though you can port-forward the service to localhost like `kubectl -n vault port-forward svc/vault 8200` but when you don't have the permission to that, how would you do it? Also, IMHO, we access it every time by a port-forward will be complicated and easy to make mistakes. I just want a direct way to visit that, e.g. an web url like `https://vault.abc.xyz`.

Ok, then we can set an ingress for the service. But the question is that, do you really dare to expose the UI to the internet, as there's no way to protect it(i.e. rate limit, IP ban or recaptcha). If you password is not that complicate, then one bad guy is possible to break through your wall through brutal force. So, the best approach for me to choose is adding one more middleware, like SSO. 

# Configuration

Tough I prefer to use one oauth-proxy with a few web apps behind it then I just have one place to pass the authentication, I didn't achieve it successfully. That is to say, I have to setup three oauth-proxy one by one for specific web app. 

First, you should decide what domain to use for your app. For me, that's `vault.abc.xyz`.

You should go to Github to creat one [OAuth App](https://github.com/settings/developers) and get the clientId and clientSecret. The homepage url could be anything, but the callback url should be lik `https://vault.abc.xyz/oauth2/callback`. 

Then we can create our manifests. 

Configmap: 
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: oauth2-proxy-config
  namespace: devops
data:
  oauth2_proxy.cfg: |
    http_address="0.0.0.0:4180"
    client_id=""
    client_secret=""
    cookie_secret="obfKcMRMbAKN97ogDriJ5HpJMnh1ItfIUIboRKvBLgM="
    cookie_domains = [".abc.xyz"]
    cookie_secrue = false
    whitelist_domains = [".abc.xyz"]
    email_domains=["*"]
    provider="github"
    github_org="MyOrg"
    upstreams=["file:///dev/null"]
    redirect_url = "https://vault.abc.xyz/oauth2/callback"
```

Deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oauth2-proxy
  namespace: devops
  labels:
    app: oauth2-proxy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: oauth2-proxy
  template:
    metadata:
      labels:
        app: oauth2-proxy
    spec:
      containers:
      - name: oauth2-proxy
        image: quay.io/oauth2-proxy/oauth2-proxy:v7.5.1
        args:
        - --config=/etc/oauth2-proxy/oauth2_proxy.cfg
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
        ports:
        - containerPort: 4180
          protocol: TCP
        volumeMounts:
        - name: config
          mountPath: /etc/oauth2-proxy
      volumes:
      - name: config
        configMap:
          name: oauth2-proxy-config
```

Ingress of Auth:
```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: oauth2-vault
  namespace: devops
  annotations:
    kubernetes.io/tls-acme: "true"
    cert-manager.io/cluster-issuer: letsencrypt-prod
    external-dns.alpha.kubernetes.io/exclude: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - vault.abc.xyz
      secretName: vault-tls
  rules:
  - host: vault.abc.xyz
    http:
      paths:
      - pathType: Prefix
        path: /oauth2
        backend:
          service:
            name: oauth2-proxy
            port:
              number: 4180
```

Ingress of Vault:
```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vault
  namespace: devops
  annotations:
    kubernetes.io/tls-acme: "true"
    cert-manager.io/cluster-issuer: letsencrypt-prod
    external-dns.alpha.kubernetes.io/exclude: "true"
    nginx.ingress.kubernetes.io/auth-signin: https://$host/oauth2/start?rd=$http_host$request_uri
    nginx.ingress.kubernetes.io/auth-url: https://$host/oauth2/auth
    nginx.ingress.kubernetes.io/auth-response-headers: Authorization
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - vault.abc.xyz
      secretName: vault-tls
  rules:
  - host: vault.abc.xyz
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vault-external
            port:
              number: 8200
```

Service: 
```yaml
apiVersion: v1
kind: Service
metadata:
  name: oauth2-proxy
  namespace: devops
  labels:
    app: oauth2-proxy
spec:
  ports:
    - port: 4180
      targetPort: 4180
      protocol: TCP
      name: http
  selector:
    app: oauth2-proxy
```

External Name:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: vault-external
spec:
  type: ExternalName
  externalName: vault.vault.svc.cluster.local
  ```

Kustomization: 
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: devops

resources:
  - deploy.yaml
  - svc.yaml
  - cm.yaml
  - external.svc.yaml
  - vault.ing.yaml
  - oauth2.ing.yaml
```

It took a lot of time for me to understand that I should split the auth path to oauth-proxy and the other paths(vault UI), if not there will be an infinite redirection or 500 Internal Server Error.

# References

[Welcome](https://oauth2-proxy.github.io/oauth2-proxy/)