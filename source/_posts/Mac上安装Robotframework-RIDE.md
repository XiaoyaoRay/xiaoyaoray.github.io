---
title: Mac上安装Robotframework-RIDE
copyright: true
date: 2018-04-19 15:50:29
tags:
- Mac
- Robotframework
- 测试
---

### 拷贝pth文件到指定目录

- [文件百度云链接](https://pan.baidu.com/s/1Ud2oGuTIdrj01z_SZZJkUg)，从百度云下载文件到目录`Downloads`

  ```shell
  sudo cp ~/Downloads/wxredirect.pth /Library/Python/2.7/site-packages/
  ```

  ​

### 拷贝wxPython目录到指定目录

- 在执行命令之前，请先确保你的/usr/local/lib目录是存在的，如果lib目录没有请自己创建一个:

  ```shell
  sudo mkdir /usr/local/lib
  ```

  ​

- 如果已经有lib目录就不用创建目录了，直接执行下面的语句

  ```shell
  sudo cp -r ~/Downloads/wxPython-unicode-2.8.12.1/ /usr/local/lib/wxPython-unicode-2.8.12.1/
  ```

  ​

- 拷贝完成后，确保/usr/local/lib/wxPython-unicode-2.8.12.1/目录下是bin、include、lib、share四个目录。

  这样就完成了wxPython的安装了，然后请自行完成ride的安装

### 安装Robotframework-RIED

- 执行以下命令

  ```shell
  pip install robotframework-ride
  ```

- 找到ride.py文件

  ```shell
  # which ride.py
  /Library/Frameworks/Python.framework/Versions/2.7/bin/ride.py
  ```

- 运行以下命令启动RIDE，主要是wxPython2.8.12.1需要32位的python2.7才能运行（`可以把命令写成shell脚本`）

  ```shell
  python2.7-32 /Library/Frameworks/Python.framework/Versions/2.7/bin/ride.py
  ```

- PS：简单用一个命令处理一下，在终端运行命令：

  ```shell
  defaults write com.apple.versioner.python Prefer-32-Bit -bool yes
  ride.py
  ```

### 参考

- [RF环境安装-mac-osx10.11-基础环境-安装指南](https://mp.weixin.qq.com/s?__biz=MjM5NTA3MDgyNg==&mid=2656981250&idx=1&sn=81e3df021485c663055deb963268e269&mpshare=1&scene=23&srcid=0419HYAgfYZogU41vorKxNOs%23rd)
- [Install wxPython 2.8 (For Ride) on OSX “El Capitan”](https://stackoverflow.com/questions/33134896/install-wxpython-2-8-for-ride-on-osx-el-capitan)
- [Can't import wx(Python) on Mac OS X](https://stackoverflow.com/questions/4798759/cant-import-wxpython-on-mac-os-x)
- [如何在Mac OS X上安装wxPython？](https://codeday.me/bug/20180319/146126.html)

