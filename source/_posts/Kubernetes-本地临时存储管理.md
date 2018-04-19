---
title: Kubernetes 本地临时存储管理
copyright: true
date: 2018-04-12 18:06:42
tags:
- Kubernetes
---

### 背  景

在 Kubernetes 中，我们可以管理限制计算资源，如 CPU， memeory，通过 quota 和 limitrange ，可以限制在一个 namespace 中，pod 和 container 可以使用多少的 cpu 和 memory，通过 downward api，可以把这个计算资源通过环境变量的方式注入到 container 中，供容器使用。 
但是，release 1.7 中还没有系统的管理、隔离本地临时存储的方法，只是加了一些简单的 eviction 逻辑和调度预测，这会导致很多问题，比如，一个 pod 如果没有设置本地临时存储限制，它可以占据 node 很多本地存储资源，导致其他 pod 被异常终止，无法限制一个 namespace 中本地临时存储总量等等，基于这些问题，经才云科技（Caicloud）和 Google 讨论，设计并实现了本地临时存储相关功能。

### 现  状

在 Kubernetes 1.7 版本中，我们引入了 alpha 版本的 LocalStorageCapacityIsolation feature，用来控制本地临时存储，但是设计和工作还处于初级阶段，不是很成熟。 在 1.7 版本中，我们把本地存储分为两种资源：一种是 scratch，代表的是 root 分区（ /var/lib/kubelet 和 /var/log 目录, 非系统根目录），另一种是 overlay，代表的是 runtime 分区。
**具体的逻辑是这样的：**

- scratch 资源（也就是 root 分区）用来满足 /var/lib/kubelet 目录 以及 /var/log 目录需求，被 EmptyDir 以及 container log 消费，如果 node 节点没有 overlay 资源的话， container image layers  和 writable layers 也是消费的 scratch 资源
- overlay 资源，就像上面说的，专门用来满足 container image layers 和 writable layers 的需求的。
  设计出来之后，所做的工作是：在 scheduling 部分加上了本地临时存储的判断逻辑，用来找到满足 pod 对本地存储需求的节点。
  但是，对本地存储的限制和隔离功能大部分都没有做，这些工作在 1.8 版本中都将加入，当然，还是 alpha 阶段。

### 目  标

像管理内存那样，Kubernetes 可以管理限制本地临时存储，以及通过获取 node 本地存储情况，把 pod 调度到合适的节点。

### 方  案

上面说了， release 1.7 中，把本地临时资源分成两种，分别代表两个分区。这样的话，管理起来很麻烦，实现逻辑也很复杂，而且 overlay 分区用处不是很大，所以经讨论最终决定，在 1.8 版本中我们只管理一个分区的资源（root partition），用来简化实现逻辑。 **资源名称为：**ResourceEphemeralStorage ResourceName="ephemeral-storage"。
下面我将分别从 eviction manager，scheduling ，quota，limitrange 以及 downward api 这些模块来介绍这个 feature 的具体实现。

#### 1eviction：

**本地临时存储的 eviction 逻辑总有三个：**

1. EmptyDir 的使用量超过了他的 SizeLimit，那么这个 pod 将会被驱逐；
2. Container 的使用量（log，如果没有 overlay 分区，则包括 imagefs）超过了他的 limit，则这个 pod 会被驱逐；
3. Pod 对本地临时存储总的使用量（所有 emptydir 和 container）超过了 pod 中所有container 的 limit 之和，则 pod 被驱逐。

**示例**：

![img](http://oj6ydypm2.bkt.clouddn.com/upfile_1509415538560.jpeg)

上面这个例子是 pod 使用本地临时存储实例，pod 中分别有两个容器，fooa 和 foob，他们对本地临时存储的需求分别是 10Gi 和 20Gi（request 和 limit 一样）， 然后，pod 还挂载了一个 EmptyDir volume，这个 volume 的 sizelimit 是 5Gi。

**下面三种情况下，pod 将会被系统驱逐：**

1. fooa 使用量超过 10Gi  或者 foob 使用量超过 20Gi （容器使用量是否包括 imagefs 视系统分区情况而定）；
2. myEmptyDir 使用量超过 5Gi ；
3. pod 总的使用量超过 30Gi ；

### 2 scheduling

这部分改动不大，只是把 1.7 版本中关于 overlay 的逻辑去掉，最终的逻辑和 cpu，memory 类似，找到满足 pod 需求的节点，调度过去。
**示例：**

![img](http://oj6ydypm2.bkt.clouddn.com/upfile_1509415592486.png)

Node 节点关于本地临时存储的定义：
在 node 的 allocatable 资源能够满足 pod 对本地临时存储需求的前提下，pod 才可能被调度到该 node 节点。

### 3. quota

在 quota 里面加入了对本地临时资源的管理限制，同一个 namespace 中，所有 pod 的本地临时存储总量不能超过 quota 的限额。
**示例：**

![img](http://oj6ydypm2.bkt.clouddn.com/upfile_1509415625164.png)

### 4.limitrange

同样的，加入了本地临时存储，可以对 pod 中 container 的本地临时存储设置限制以及默认值等，和 memory 类似。
**示例：**

![img](http://oj6ydypm2.bkt.clouddn.com/upfile_1509415659430.png)

如果在该 namespace 中，pod 没有对本地存储进行 request/limit 定义，limitrange 会对 pod 进行默认赋值，结果类似下面：

![img](http://oj6ydypm2.bkt.clouddn.com/upfile_1509415674161.png)

### 5downward api

同样的，实现了本地存储的这个功能，可以通过环境变量的方式，把 pod 中某个 container 的本地临时存储需求 注入到特定的容器中。
**示例：**

![img](http://oj6ydypm2.bkt.clouddn.com/upfile_1509415695807.jpeg)

### 将  来

本地临时存储还处于初级（alpha）阶段。后面，Caicloud 和 Google 将继续合作，在 1.9 版本中完善一些功能，把它推进到 beta 版本，之后越来越完善。
其他的示例都类似，可以参考下面链接https://github.com/kubernetes/community/pull/991https://github.com/kubernetes/community/pull/306

### 参考

[Kubernetes 1.8 新进展！将实现本地临时存储管理](http://www.k8smeetup.com/article/VyEncpgA7)