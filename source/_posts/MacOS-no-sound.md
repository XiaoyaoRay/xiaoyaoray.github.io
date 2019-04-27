---
title: macOS没有声音
date: 2019-02-21 14:07:40
categories:
- MacOS
tags:
- MacOS
---

### 睡眠状态恢复之后没有声音

有时候 Mac 从睡眠状态恢复之后没有声音，这是 Mac OS X 系统的一个 Bug。这是因为 Mac OS X 的核心音频守护进程「coreaudiod」出了问题，虽然简单的重启电脑就能解决，但是如果此时开启了很多程序后者有其他情况不想重启电脑的话，可以按照下面的方法解决此问题。
　　操作步骤：
　　1、在 Mac 中打开活动监视器（在 Finder 的`应用程序`中搜索`活动监视器`可以找到）

​	![image-20190221141031724](https://ws4.sinaimg.cn/large/006tKfTcgy1g0e0kx00iyj30ai08kdhz.jpg)

　　2、在「活动监视器」窗口右上角的搜索框里输入`coreaudiod`，搜索到进程![image-20190221141126697](https://ws4.sinaimg.cn/large/006tKfTcgy1g0e0ltf69vj327808ajvd.jpg)

　　3、选中「coreaudiod」进程，点击「活动监视器」窗口左上角的「退出进程」按钮，在弹出的对话框中点击「退出」![image-20190221141251203](https://ws3.sinaimg.cn/large/006tKfTcgy1g0e0nak1i7j313g082wha.jpg)
　　4、「coreaudiod」进程退出后会自动重启，这时声音就恢复了。