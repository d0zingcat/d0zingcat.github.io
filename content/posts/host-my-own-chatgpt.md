---
author: ["Me"]
title: 'Host My Own ChatGPT'
date: 2024-11-29T20:59:49+08:00
categories: ["AI", "Self-Hosted"]
tags: ["chatgpt", "openai", "openrouter", "kubernetes"]
draft: false
---

The Claude is quite difficult to get onboard, as you have to prepare a phone number and a credit card. Also, if you subscribe to the Pro plan, you will only get access the the Claude model(it's quite powerful). But I prefer more choices, I want to use different models under different circumstances, e.g. I want to use Claude when I am coding, use OpenAI o1 to to math or reasoning or gpt-3.5-turbo to do translation. On this point, I use [OpenRouter](https://openrouter.ai/). You can top up your balance by your Crypto(like USDC), which takes less tax fees.

You can find the latest/popular models here: [Rankings](https://openrouter.ai/rankings/programming?view=week), 
for programming, there's no double that people are using claude/sonnet-3.5 mostly. 

You can use OpenRouter's web interface to chat [here](https://openrouter.ai/chat). 
It's quite simple and convenient to use. You can select a few models to watch the differences between them. 
However, it does not support a predefined prompt. So I choose to host my own web UI. I decide to use [ChatgptNext](https://github.com/ChatGPTNextWeb/ChatGPT-Next-Web/tree/main). It's quite simple to use and supports a bunch of predefined prompts. Also, it supports artifacts and Authtication. 

I am using helm and my own base-app definition to install it. Maybe one day I will share this chart with when it's full-fledged. Before that, you can use [this library](https://github.com/bjw-s/helm-charts).

Let me share my values to you:

```yaml
replicaCount: 1

image:
  repository: yidadaa/chatgpt-next-web
  pullPolicy: IfNotPresent
  tag: "latest"

imagePullSecrets: []
nameOverride: "chatgpt-next"
fullnameOverride: "chatgpt-next"

service:
  ports:
  - type: ClusterIP
    port: 3000
    name: http
    containerPort: 3000

ingress:
  enabled: true
  className: "nginx"
  annotations: 
    external-dns.alpha.kubernetes.io/hostname: chatgpt-next.d0zingcat.dev
    cert-manager.io/cluster-issuer: letsencrypt-prod
    kubernetes.io/tls-acme: "true"
  hosts:
    - host: chatgpt-next.d0zingcat.dev
      tls: chatgpt-next.d0zingcat.dev-tls
      paths:
        - path: /

env: 
- name: OPENAI_API_KEY
  value: "my-openrouter-api-key"
- name: BASE_URL
  value: "https://openrouter.ai/api"
- name: CODE
  value: MyP@ssw0rd
- name: ENABLE_BALANCE_QUERY
  value: "1"
- name: CUSTOM_MODELS
  value: >-
    -all,
    openai/gpt-4o-mini,openai/gpt-4o,openai/chatgpt-4o-latest,openai/o1-preview,openai/o1-mini,
    anthropic/claude-3.5-sonnet:beta,anthropic/claude-3.5-sonnet,anthropic/claude-3-haiku,anthropic/claude-3-opus,anthropic/claude-3-opus:beta,
    google/gemini-flash-1.5-8b,google/gemini-flash-1.5,google/gemini-flash-1.5-exp,google/gemini-pro-1.5,google/gemini-pro-1.5-exp,google/gemma-2-27b-it,google/gemma-2-9b-it,
    meta-llama/llama-3.1-70b-instruct,meta-llama/llama-3.1-8b-instruct,meta-llama/llama-3-70b-instruct,meta-llama/llama-3-8b-instruct,
    qwen/qwen-2.5-coder-32b-instruct,qwen/qwq-32b-preview,qwen/qwen-2-vl-72b-instruct,qwen/qwen-2.5-72b-instruct,
    deepseek/deepseek-chat,
    mistralai/mistral-nemo,mistralai/mistral-tiny,mistralai/mistral-7b-instruct,cohere/command-r,gryphe/mythomax-l2-13b,microsoft/wizardlm-2-8x22b,cohere/command-r-08-2024,nousresearch/hermes-3-llama-3.1-405b:free,microsoft/wizardlm-2-7b,mistralai/mistral-7b-instruct-v0.2
- name: DEFAULT_MODEL
  value: gpt-4o-mini
```

To be noted, I define following things:

- API_KEY
- API_BASE_URL
- CODE to get authenticated
- CUSTOM_MODELS(for OpenRouter API), I've picked some popular models
- DEFAULT_MODEL for default use

You can visit my webpage for your personal use, but you have to supply your own API_KEY to get started(as you do not have a CODE to use my API_KEY). 


## References

[OpenRouter Integration #3361](https://github.com/ChatGPTNextWeb/ChatGPT-Next-Web/discussions/3361)