---
title: Python代码获取系统环境变量的值
date: 2018-09-26 14:31:45
categories:
- Python
tags:
- Python
---

### 转载：[python 获取系统环境变量 os.environ and os.putenv](https://www.cnblogs.com/brownz/p/8360292.html)

### 设置系统环境变量

```
import os

os.environ['环境变量名称']='环境变量值' #其中key和value均为string类型
os.putenv('环境变量名称', '环境变量值')
```

 <!--more-->

### 获取系统环境变量

```
import os

os.environ['环境变量名称']
os.getenv('环境变量名称')
```