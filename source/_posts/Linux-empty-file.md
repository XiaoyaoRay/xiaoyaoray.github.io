---
title: Linux下置空文件的方法小结
date: 2018-12-26 14:16:05
categories:
- Linux
tags:
- Linux
- Shell
---

## 转载：[Linux下置空文件的方法小结](https://blog.csdn.net/wangjianno2/article/details/39755805)

### 彻底置空

就是ls文件的大小为0，文件里面什么都没有

     （1）: > filename
     （2）true > filename
     （3）cat /dev/null > filename
     （4）> filename

<!--more-->

### 置空文件

文件中有空行，ls文件的大小，显示还有大小

    （1）echo "" > filename
    （2）echo > filename
