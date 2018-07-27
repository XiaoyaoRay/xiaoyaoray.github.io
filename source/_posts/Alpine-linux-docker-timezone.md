---
title: 修改使用Alpine Linux的Docker容器的时区
copyright: true
date: 2018-04-27 16:01:48
tags:
- Docker
- Alpine
- Linux
---

#### 适用对象

- 使用 Alpine Linux 发行版的 Docker 镜像容器。
- 仅仅适用于**没有**安装`uclibc`的系统。
<!--more-->
#### 修改步骤

- 进入容器命令行

  ```
  ＃ docker exec -it container_name /bin/sh
  ```

- 安装 timezone 数据包

  ```
  ＃ apk add -U tzdata
  ＃ ls /usr/share/zoneinfo
  ```

  为了防止添加失败，加上`-U`参数，更新仓储缓存。
  列出安装的时区文件，验证是否下载成功。

- 拷贝需要的时区文件到`localtime`，国内需要的是`Asia/Shanghai`：

  ```
  ＃ cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
  ```

- 验证时区

  ```
   ＃ date
   Tue Jan  9 22:53:46 CST 2018
  ```

  `CST` 即为 `中国标准时间`。

- 移除时区文件：

  ```
  ＃ apk del tzdata
  ```

  为了保证容器的精简和轻量，移除下载的时区文件。

#### 参考资料：

[https://wiki.alpinelinux.org/wiki/Setting_the_timezone](https://link.jianshu.com?t=https%3A%2F%2Fwiki.alpinelinux.org%2Fwiki%2FSetting_the_timezone)

https://www.jianshu.com/p/cd1636c94f9f
