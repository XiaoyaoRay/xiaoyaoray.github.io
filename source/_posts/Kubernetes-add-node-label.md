---
title: K8S对node添加label，并根据label筛选节点
date: 2018-12-26 11:46:59
categories:
- Kubernetes
tags:
- Kubernetes
---

## 转载：[K8S对node添加label，并根据label筛选节点](https://blog.csdn.net/gsying1474/article/details/59057519)

某些特殊情况下，需要将某些服务固定在一台宿主机上，K8S也适应这种方式，下面以mongo为例，来看看如何实现的：

```kubectl label nodes kube-node node=kube-node
kubectl label nodes kube-node node=kube-node
kubectl get node -a -l "node=kube-node"
```


pod或者rc的配置项中添加如下配置：

```
nodeSelector: 
       node: kube-node4
```

<!--more-->

如mongo启动的rc文件:

```
apiVersion: v1
kind: ReplicationController
metadata:
 name: mongo
spec:
 replicas: 1 
 template:
   metadata:
     labels:
       run: mongo
   spec:
     containers:
     - name: mongo
       image: daocloud.io/library/mongo:3.2.4
       ports:
         - containerPort: 27017
       volumeMounts:
         - mountPath: /data/db
           name: mongo
     volumes: [{"name":"mongo","hostPath":{"path":"/root/volumes/mongo"}}]
     nodeSelector: 
       node: kube-node4
```