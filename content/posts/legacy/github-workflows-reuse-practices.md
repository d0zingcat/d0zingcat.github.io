---
author: ["Me"]
title: 'Github Workflow Reuse Practice'
date: 2024-11-20T15:24:55+08:00
categories: []
tags: []
draft: true
---

```
name: build-publish

on:
  workflow_call: 
    inputs: 
      image_name:
        required: true
        type: string
      image_tags:
        required: true
        type: string
      notify_channel:
        required: false
        type: string
      argocd_app_name:
        required: false
        type: string
      dockerfile:
        required: false
        type: string
      docker_build_args:
        required: false
        type: string

env:
  REGISTRY: ghcr.io

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Check if user has write access
        uses: lannonbr/repo-permission-check-action@2.0.2
        with:
          permission: write
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Notify Starting
        if: ${{ inputs.notify_channel != '' && success() }}
        id: slack # IMPORTANT: reference this step ID value in future Slack steps
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
        uses: devopsATmeshee/github-action-slack-notify-build@main
        with:
          channel: ${{ inputs.notify_channel }}
          status: STARTING
          color: warning
          comment: "Image: ${{ inputs.image_name }}"

      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log into registry ${{ env.REGISTRY }}
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ inputs.image_name }}
          tags: ${{ inputs.image_tags }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ./
          file: ${{ inputs.dockerfile != '' && inputs.dockerfile || 'Dockerfile' }}
          push: ${{ github.event_name != 'pull_request' }}
          build-args: ${{ inputs.docker_build_args }}

          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Notify Packed
        if: ${{ inputs.notify_channel != '' && success() }}
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
        uses: devopsATmeshee/github-action-slack-notify-build@main
        with:
          message_id: ${{ steps.slack.outputs.message_id }}
          channel: ${{ inputs.notify_channel }}
          status: PACKED
          color: good
          comment: "Image: ${{ inputs.image_name }}"

      - name: Update ArgoCD Image
        if: ${{ inputs.argocd_app_name != '' }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ARGOCD_VERSION: v2.6.7
        run: |
          set -ex
          # download and install argocd cli
          curl -sSL -o /tmp/argocd-${ARGOCD_VERSION} https://github.com/argoproj/argo-cd/releases/download/${ARGOCD_VERSION}/argocd-linux-amd64
          chmod +x /tmp/argocd-${ARGOCD_VERSION}
          sudo mv /tmp/argocd-${ARGOCD_VERSION} /usr/local/bin/argocd
          # iter and update one by one
          while IFS= read -r app; do
            if [ -z "$app" ]; then
              echo "Skipping empty or blank app name"
              continue
            fi
            echo "Updating App: $app"
            argocd app set $app \
            --server "${{ vars.ARGOCD }}" \
            --kustomize-image "${{ fromJSON(steps.meta.outputs.json).tags[0] }}" \
            --auth-token "${{ secrets.ARGOCD_TOKEN }}" \
            --loglevel debug \
            --grpc-web
            # TODO: redo for temp
            argocd app set $app \
            --server "${{ vars.ARGOCD }}" \
            --kustomize-image "${{ fromJSON(steps.meta.outputs.json).tags[0] }}" \
            --auth-token "${{ secrets.ARGOCD_TOKEN }}" \
            --loglevel debug \
            --grpc-web
          done <<< "${{ inputs.argocd_app_name }}"

      - name: Notify Published
        if: ${{ inputs.notify_channel != '' && inputs.argocd_app_name != '' && success() }} 
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
        uses: devopsATmeshee/github-action-slack-notify-build@main
        with:
          message_id: ${{ steps.slack.outputs.message_id }}
          channel: ${{ inputs.notify_channel }}
          status: PUBLISHED
          color: good
          comment: "Image: ${{ inputs.image_name }}"
```