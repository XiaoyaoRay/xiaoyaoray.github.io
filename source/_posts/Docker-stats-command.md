---
title: Docker stats监控容器资源消耗
date: 2019-01-15 17:16:34
categories:
- Docker
tags:
- Docker
---

## 转载：[docker stats监控容器资源消耗](https://blog.csdn.net/hu_jinghui/article/details/80198492)

在容器的使用过程中，如果能及时的掌握容器使用的系统资源，无论对开发还是运维工作都是非常有益的。幸运的是 docker 自己就提供了这样的命令：docker stats。



### 默认输出

docker stats 命令用来显示容器使用的系统资源。不带任何选项执行 docker stats 命令：

```
`$ docker stats`
```

![image-20190115171844942](https://ws2.sinaimg.cn/large/006tNc79gy1fz7e3d8j4uj30s70cn41l.jpg)

默认情况下，stats 命令会每隔 1 秒钟刷新一次输出的内容直到你按下 ctrl + c。下面是输出的主要内容：

```
[CONTAINER]：以短格式显示容器的 ID。
[CPU %]：CPU 的使用情况。
[MEM USAGE / LIMIT]：当前使用的内存和最大可以使用的内存。
[MEM %]：以百分比的形式显示内存使用情况。
[NET I/O]：网络 I/O 数据。
[BLOCK I/O]：磁盘 I/O 数据。 
[PIDS]：PID 号。
```



### 只返回当前的状态

如果不想持续的监控容器使用资源的情况，可以通过 --no-stream 选项只输出当前的状态：

`$ docker stats --no-stream`

这样输出的结果就不会变化了，看起来省劲不少。

![image-20190115172320116](https://ws1.sinaimg.cn/large/006tNc79gy1fz7e82q348j30ss06l401.jpg)

### 只输出指定的容器

如果我们只想查看个别容器的资源使用情况，可以为 docker stats 命令显式的指定目标容器的名称或者是 ID：

`$ docker stats --no-stream registry 1493`

![image-20190115172402634](https://ws2.sinaimg.cn/large/006tNc79gy1fz7e8thgwcj30sm02bdg8.jpg)

当有很多的容器在运行时，这样的结果看起来会清爽一些。这里的 `baa9c5417cab` 和 `629644309f60` 分别是容器的名称和容器的 ID。注意，多个容器的名称或者是 ID 之间需要用空格进行分割。

细心的同学可能已经发现了，第一列不再显示默认的容器 ID，而是显示了我们传入的容器名称和 ID。基于此，我们可以通过简单的方式使用容器的名称替代默认输出中的容器 ID：

`$ docker stats $(docker ``ps` `--``format``={{.Names}})`

![image-20190115172453381](https://ws1.sinaimg.cn/large/006tNc79gy1fz7e9pbqybj31e704qjsm.jpg)

用容器的名称替代 ID 后输出的结果是不是友好一些？

### 格式化输出的结果

我们在前面搞了点小手段把输出中的容器 ID 替换成了名称。其实 docker stats 命令支持我们通过 --format 选项自定义输出的内容和格式：

`$ docker stats --``format` `"table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"`

![image-20190115173937378](https://ws2.sinaimg.cn/large/006tNc79gy1fz7ep0fozvj30zh03qdh3.jpg)

上面的命令中我们只输出了 `Name`, `CPUPerc` 和 `Memusage` 三列。下面是自定义的格式中可以使用的所有占位符：

```
.Container    根据用户指定的名称显示容器的名称或 ID。
.Name           容器名称。
.ID                 容器 ID。
.CPUPerc       CPU 使用率。
.MemUsage  内存使用量。
.NetIO           网络 I/O。       
.BlockIO        磁盘 I/O。
.MemPerc     内存使用率。
.PIDs             PID 号。
```

有了这些信息我们就可以完全按照自己的需求或者是偏好来控制 docker stats 命令输出的内容了。

除了以 table 格式输出结果，还可以通过 format 选项输出 json 格式的结果：

`$ docker stats --no-stream --format \``"{\"container\":\"{{ .Container }}\",\"memory\":{\"raw\":\"{{ .MemUsage }}\",\"percent\":\"{{ .MemPerc }}\"},\"cpu\":\"{{ .CPUPerc }}\"}"`

![image-20190115174154514](https://ws3.sinaimg.cn/large/006tNc79gy1fz7erem25yj317h033dgz.jpg)

**总结**

通过 docker stats 命令我们可以看到容器使用系统资源的情况。这为我们进一步的约束容器可用资源或者是调查与资源相关的问题提供了依据。除了 docker 自带的命令，像 glances 等工具也已经支持查看容器使用的资源情况了，有兴趣的朋友可以去了解一下。