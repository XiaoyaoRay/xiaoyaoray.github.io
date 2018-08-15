---
title: Kubernetes没有拉取镜像权限
copyright: true
date: 2018-06-12 15:47:11
categories:
- Kubernetes
tags:
- Kubernetes
- Wise2c
---

https://kubernetes.io/docs/concepts/configuration/secret/#creating-a-secret-manually
<!--more-->
```
# 获取系统中默认的secret，并把yaml拷贝出来
kubectl get secret
kubectl get secret privateregistrykey -o yaml > xxx.yaml

# 编辑yaml文件修改namespace
vim xxx.yaml 
namespace: ns-team-1-env-11

# 新建secret，并配置
kubectl create -f xxx.yaml
kubectl -n ns-team-1-env-11 patch serviceaccount default -p '{"imagePullSecrets":[{"name": "privateregistrykey"}]}'
```

