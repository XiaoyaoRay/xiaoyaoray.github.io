---
title: Pip使用
copyright: true
date: 2018-08-10 14:12:47
categories:
- Python
tags:
- Python
---

## Pip 安装

### CentOS

```
# 安装epel扩展源
yum -y install epel-release

# 然后再安装pip
yum -y install python-pip

# 更新pip
pip install --upgrade pip
```

## Pip更换国内镜像源

阿里云 <http://mirrors.aliyun.com/pypi/simple/> 
中国科技大学 <https://pypi.mirrors.ustc.edu.cn/simple/> 
豆瓣(douban) <http://pypi.douban.com/simple/> 
清华大学 <https://pypi.tuna.tsinghua.edu.cn/simple/> 
中国科学技术大学 <http://pypi.mirrors.ustc.edu.cn/simple/>

<!--more-->

### Pip源配置文件存放路径

不同系统的配置路径可能不同

#### Linux/Unix

```
/etc/pip.con
~/.pip/pip.conf  （每一个我都找了都没有，所以我是在这个文件夹中创建的pip.conf文件）
~/.config/pip/pip.conf 
```

#### Mac OSX

```
~/Library/Application Support/pip/pip.conf
~/.pip/pip.conf
/Library/Application Support/pip/pip.conf 
```

#### Windows

```
%APPDATA%\pip\pip.ini
%HOME%\pip\pip.ini
C:\Documents and Settings\All Users\Application Data\PyPA\pip\pip.conf (Windows XP)
C:\ProgramData\PyPA\pip\pip.conf (Windows 7及以后)
```

### 修改配置文件

#### 临时使用

可以在使用pip的时候在后面加上-i参数，指定pip源

```
pip install scrapy -i <https://pypi.tuna.tsinghua.edu.cn/simple>
```

#### 永久修改

如果没有需要自己创建一个，一般使用该路径`~/.pip/pip.conf`

```
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/ #豆瓣源，可以换成其他的源         
disable-pip-version-check = false          #如果为true则取消pip版本检查，排除每次都报最新的pip
timeout = 120

[install]
trusted-host = mirrors.aliyun.com	#添加豆瓣源为可信主机，要不然可能报错
```

## Pip使用详解

### Pip安装包

```
# pip install SomePackage
  [...]
  Successfully installed SomePackage
```

### Pip查看已安装的包

```
# pip show --files SomePackage
  Name: SomePackage
  Version: 1.0
  Location: /my/env/lib/pythonx.x/site-packages
  Files:
   ../somepackage/__init__.py
   [...]
```

### Pip检查哪些包需要更新

```
# pip list --outdated
  SomePackage (Current: 1.0 Latest: 2.0)
```

### Pip升级包

```
# pip install --upgrade SomePackage
  [...]
  Found existing installation: SomePackage 1.0
  Uninstalling SomePackage:
  Successfully uninstalled SomePackage
  Running setup.py install for SomePackage
  Successfully installed SomePackage
```

### Pip卸载包

```
$ pip uninstall SomePackage
  Uninstalling SomePackage:
    /my/env/lib/pythonx.x/site-packages/somepackage
  Proceed (y/n)? y
  Successfully uninstalled SomePackage
```

### Pip参数解释

```
# pip --help
 
Usage:   
  pip <command> [options]
 
Commands:
  install                     安装包.
  uninstall                   卸载包.
  freeze                      按着一定格式输出已安装包列表
  list                        列出已安装包.
  show                        显示包详细信息.
  search                      搜索包，类似yum里的search.
  wheel                       Build wheels from your requirements.
  zip                         不推荐. Zip individual packages.
  unzip                       不推荐. Unzip individual packages.
  bundle                      不推荐. Create pybundles.
  help                        当前帮助.
 
General Options:
  -h, --help                  显示帮助.
  -v, --verbose               更多的输出，最多可以使用3次
  -V, --version               现实版本信息然后退出.
  -q, --quiet                 最少的输出.
  --log-file <path>           覆盖的方式记录verbose错误日志，默认文件：/root/.pip/pip.log
  --log <path>                不覆盖记录verbose输出的日志.
  --proxy <proxy>             Specify a proxy in the form [user:passwd@]proxy.server:port.
  --timeout <sec>             连接超时时间 (默认15秒).
  --exists-action <action>    Default action when a path already exists: (s)witch, (i)gnore, (w)ipe, (b)ackup.
  --cert <path>               证书.
```

### 参考

[python pip源配置,pip配置文件存放位置](https://blog.csdn.net/u013066730/article/details/54580789)

[更换pip源到国内镜像](https://blog.csdn.net/chenghuikai/article/details/55258957)

[pip安装使用详解](https://blog.csdn.net/u013066730/article/details/54580863)
