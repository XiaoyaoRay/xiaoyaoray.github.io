---
title: Docker清理方案
copyright: true
date: 2018-06-21 16:41:20
tags:
- Docker
---

## 清理overlay存储

我们在使用docker的过程中发现基于swarm使用Storage Driver: overlay的方式进行存储.但是发现这个特别占用存储空间.
<!--more-->
### 清理所有停止的容器

```
docker container prune 
```

### 清理所有不用数据(停止的容器,不使用的volume,不使用的networks,*悬挂*的镜像)

```
docker system prune -a
```

### overlay存储

我们通过上面的操作清除了一些无用的数据,但是,overlay还是特别大.我们先了解下overlay存储.

#### overlayfs

集成进了linux 3.18内核.
 overlay存储驱动主要使用的是overlayfs技术.中文名是叠合式文件系统.多个文件系统可以mount之后进行合并.
 docker 镜像层  lowerdir
 docker 容器层  upperdir
 docker 容器挂载点 merged

这个三个层对应了 overlayFs的结构. 我们通过docker inspect 可以查看到如下结构

```
"GraphDriver": {
            "Name": "overlay",
            "Data": {
                "LowerDir": "/mnt/docker/overlay/5eb97eb91bed89a9c879142900419ad118215af05c291989282c130d031d7019/root",
                "MergedDir": "/mnt/docker/overlay/454f70c61de03ce2a517d7e2ea8c19e319a95cd2275d8b826f4244071315e513/merged",
                "UpperDir": "/mnt/docker/overlay/454f70c61de03ce2a517d7e2ea8c19e319a95cd2275d8b826f4244071315e513/upper",
                "WorkDir": "/mnt/docker/overlay/454f70c61de03ce2a517d7e2ea8c19e319a95cd2275d8b826f4244071315e513/work"
            }
        }
```

镜像在 /root
 挂载点在 /merged
 容器在 /upper
 工作目录 /work

#### overlayfs数据清理

我们做了一个实验,我们启动一个容器(版本不同),之后

```
docker stop conatiner
docker rm container
```

通过对数据大小的监控,我们发现 overlay会随着新镜像的产生而产生一些数据,随着容器的关闭删除,这个文件并没有缩小体积.如何解决呢? 看来我们忽视了一个问题.我们使用 `docker system prune`以为可以不需要的数据都清理了,但是关于images中是这样描述的" dangling images" 悬挂的镜像 .关于这个词汇我还没有理解.不过通过测试,即使我把容器停止也无法清理镜像,所以,我无法理解 悬挂是怎样的状态.

```
docker rmi images
```

最后我们通过手动删除镜像,则之前产生的overlay数据就随之减少了.

*tips:*
 查看overlay 大小 `du --max-depth=1 -h`
 查看数量 `ls|wc -w`

### 整理

镜像有新版本产生的话,我们可以按照这样的流程操作

```
docker stop container
docker rm container
docker rmi image
docker pull image
docker run ...
```

这样就避免了系统磁盘一眨眼的时间就满了.

### 其他

#### docker 时间同步?

```
ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo Asia/Shanghai > /etc/timezone
```

#### docker 存储地方修改

```
ExecStart=/usr/bin/dockerd --graph="/mnt/data/images"
```

 

## 清理容器日志

 docker 用久了 日志一大堆，很占用空间，不用的日志可以清理掉了。 
docker logs -f container name 噼里啪啦 一大堆，，，，太对，清理掉

### 第一步日志位置

找到对应container的日志文件，一般是在 `/var/lib/docker/containers/containerid/containerid.log-json.log`（`containerid`是指你的容器id）

#### 找日志位置

如果找不到，可以模糊查询一下 `find / -type f -name "*.log" | xargs grep "ERROR"` 找到日志位置(这行命令的意思是从根目录开始查找所有扩展名为.log的文本文件，并找出包含”ERROR”的行，你可把 error 换成你日志中存在的内容，docker logs -f container name 就能看到有什么内容啦)

#### 找容器id

如果不知道容器id是什么， docker inspect Container name 可以看到容器id

### 第二部：清理一下

```
cat /dev/null >/var/lib/docker/containers/containerid/<containerid>.log-json.log
```

 

 
