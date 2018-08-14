---
title: centos中安装docker-compose
copyright: true
date: 2018-08-14 09:58:00
tags:
- Docker-compose
---

## 安装

### 方法一

[官网安装](https://github.com/docker/compose/releases/)

找到需要安装的版本，用curl下载进行安装（以版本`1.22.0`为例）

```
curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

查看版本信息

```
docker-compose --version
```

<!--more-->

### 方法二

用pip来安装，如果没pip命令请先安装pip

```
yum -y install epel-release
yum -y install python-pip
```

安装docker-compose

```
pip install docker-compose
# 待安装完成后，执行查询版本的命令，即可安装docker-compose
docker-compose --version
```

## 卸载

如果用 `curl`安装的，运行以下命令:

```
sudo rm /usr/local/bin/docker-compose
```

如果用 `pip`安装的，运行以下命令:

```
pip uninstall docker-compose
```

## 参考

[安装docker-compose的两种方式](https://blog.csdn.net/gsying1474/article/details/52988784)

[官网文档](https://docs.docker.com/compose/install/#uninstallation)



