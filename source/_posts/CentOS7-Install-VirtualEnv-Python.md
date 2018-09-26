---
title: CentOS7安装python虚拟运行环境
copyright: true
date: 2018-05-30 14:45:55
categories:
- Python
tags:
- CentOS
- Python
---

#### 安装PIP

```
[root@ray ~]# yum -y install python-pip
```
<!--more-->
#### 安装Virtualenv

```
[root@ray ~]# pip install virtualenv
[root@ray ~]# virtualenv --version
```

#### 安装新的Python版本

- 安装编译需要的包

  ```
  [root@ray ~]# yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
  ```

- 下载python源码

  ```
  [root@ray ~]# wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tar.xz
  ```

- 编译源码

  ```
  [root@ray ~]# tar -xvjf Python-3.6.5.tar.xz
  [root@ray ~]# cd Python-3.6.5
  [root@ray ~]# ./configure prefix=/usr/local/python3
  [root@ray ~]# make & make install
  ```

#### 创建虚拟环境

```
[root@ray pythonenv]# virtualenv -p /usr/local/python3/bin/python3.6 python36
Running virtualenv with interpreter /usr/local/python3/bin/python3.6
Using base prefix '/usr/local/python3'
New python executable in /pythonenv/python36/bin/python3.6
Also creating executable in /pythonenv/python36/bin/python
Installing setuptools, pip, wheel...done.

# 启动python
[root@ray python36]# source bin/activate
(python36) [root@ray python36]#
```

#### 参考

https://blog.csdn.net/blueheart20/article/details/70598031

https://www.cnblogs.com/JahanGu/p/7452527.html
