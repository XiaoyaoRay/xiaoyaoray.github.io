---
title: Linux添加定时任务
copyright: true
date: 2018-04-27 16:04:59
tags:
- Linux
---

## 1. 使用 crontab -e 命令编辑定时任务列表

使用这个命令编辑的定时任务列表是属于用户级别的，初次编辑后在 /var/spool/cron 目录下生成一个与用户名相同的文件，文件内容就是我们的定时任务列表。如没有定时任务，这个文件就是空文件。

crontab命令还有一些其他的选项

```
-u 	#指定哪个用户的cron服务，一般是root用户执行这个命令的时候需要
-l 	#列出用户的定时任务列表，默认当前用户
-r 	#删除用户的定时任务列表，默认当前用户 
-e 	#编辑用户的定时任务列表，默认当前用户　　
```

<!--more-->
例子：列出xiaoming用户的cron服务列表

```
crontab -u xiaoming -l
```

 

## 2. 直接编辑 /etc/crontab 文件，命令如下：

编辑 /etc/crontab 文件只有 root 用户才行

```
vim /etc/crontab
```

我们会看到文件内容，如下：

```
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
```

这配置的定时任务属于系统级别的。

 

## 3. 其他的一些区别

crontab -e 会进行语法检查、直接编辑 /etc/crontab 文件则不会

### 参考

 [Linux 添加定时任务，crontab -e 命令与直接编辑 /etc/crontab 文件](http://www.cnblogs.com/libra0920/p/6520701.html)
