---
title: Apline Linux 系统简介
copyright: true
date: 2018-08-10 14:05:06
categories:
- Linux
tags:
- Apline
---

## 转载：[Apline Linux](https://www.jianshu.com/p/2c1118694936)

Apline Linux是一个面向安全应用的轻量级Linux发行版。它采用了musl libc和busybox以减小系统的体积和运行时资源的消耗，同时还提供了自己包管理工具apk;

Apline Linux的内核都打上了grsecurity/Pax补丁，并且所有的程序都编译为Postion Independent Executables(PIE)以增强系统的安全性

<!--more-->

## Apline Linux的优缺点

### 优点

- Apline Linux的Docker镜像特点就是轻巧(大小只有5M)且有完整的包管理工具，需要更少的运行资源；

### 缺点

- Apline Linux使用了musl libc，可能和其他发行版GUN/Linux都使用的glibc实现会有些不同；

## Apline Linux包管理

Apline Linux使用apk进行包管理，通过apk --help命令查看完整的管理命令，下面列举常用的命令：

升级包

```
apk update  # 更新最新本地镜像源
apk upgrade # 升级软件
apk add --upgrade busybox # 指定升级部分软件包
```

安装包

```
apk add openssh vim
apk add --no-cache mysql-clinet
apk add docker --update-cache --repository http://mirrors.ustc.edu.cn/alpine/v3.4/main/ --allow-untrusted
```

卸载删除包

```
apk del xxx
```

search搜索可用的包

```
apk search 
apk search -v 'acf*' # 通过包名搜索
apk search -v -d 'docker' # 通过描述文件搜索
```

info显示包信息

```
apk info # 列出所有已安装的包
apk info -a zlib # 显示完整的软件包信息
apk info --who-owns /sbin/lbu # 显示指定文件属于的包
```

## Apline镜像源

国内镜像源

```
清华TUNA镜像源：https://mirror.tuna.tsinghua.edu.cn/alpine/
中科大镜像源：http://mirrors.ustc.edu.cn/alpine/
阿里云镜像源：http://mirrors.aliyun.com/alpine/
```

配置镜像源

以中科大源为例：在/etc/apk/repositories文件中加入对应源地址就行了，一行一个地址。

```
$ vi /etc/apk/repositories
# /media/cdrom/apks
http://mirrors.ustc.edu.cn/alpine/v3.5/main
http://mirrors.ustc.edu.cn/alpine/v3.5/community
```

## Alpine Linux init系统

[https://wiki.alpinelinux.org/wiki/Alpine_Linux_Init_System](https://link.jianshu.com?t=https://wiki.alpinelinux.org/wiki/Alpine_Linux_Init_System)

Alpine Linux使用的是Gentoo一样的OpenRC init系统, 需要注意的是，docker apline并没有启动rc init, 如果需要使用, `apk add openrc --no-cache`;

- rc-update: 主要用于不同运行级增加或者删除服务

  ```
  rc-update add docker boot #增加一个服务
  rc-update del docker boot #删除一个服务
  ```

- rc-status: 主要用于运行级的状态管理

  ```
  rc-status #检查默认运行级别的状态
  rc-status -a #检查所有运行级别的状态
  ```

- rc-service: 主用于管理服务的状态

  ```
  rc-service sshd start #启动一个服务。
  rc-service sshd stop #停止一个服务。
  rc-service sshd restart #重启一个服务。
  ```

可用的运行级别

- sysinit - Brings up any system specific stuff such as /dev, /proc and optionally /sys for Linux based systems. It also mounts /lib/rc/init.d as a ramdisk using tmpfs where available unless / is mounted rw at boot. rc uses /lib/rc/init.d to hold state information about the services it runs. sysinit always runs when the host first starts and should not be run again.
- boot - Generally the only services you should add to the boot runlevel are those which deal with the mounting of filesystems, set the initial state of attached peripherals and logging. Hotplugged services are added to the boot runlevel by the system. All services in the boot and sysinit runlevels are automatically included in all other runlevels except for those listed here.
- single - Stops all services except for those in the sysinit runlevel.
- reboot - Changes to the shutdown runlevel and then reboots the host.
- shutdown - Changes to the shutdown runlevel and then halts the host.

 

 

 

 

 
