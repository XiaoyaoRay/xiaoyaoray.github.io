---
title: Mac OS 操作 
copyright: true
tags:
- Mac
---

### MAC OS上配置Lantern代理

1. 首先确保设备在同一个局域网内

2. 先关闭你的 Lantern，然后通过命令行启动 Lantern，在电脑终端输入：
<!--more-->
   ```shell
   /Applications/Lantern.app/Contents/MacOS/lantern -addr 0.0.0.0:8787 
   ```

3. 找到Mac的IP地址，在需要代理的机器上配置<mac-ip>:8787

### MAC OS上获取Finder中文件路径

- 打开终端，输入下面的命令，Finder就能在顶部看见完整的地址了

  ```shell
  defaults write com.apple.finder _FXShowPosixPathInTitle -bool YES
  ```


- 在Finder里，对选中的文件，command+option+c，复制到剪贴板

