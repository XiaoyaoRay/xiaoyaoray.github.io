---
title: Python中环境变量的查看与添加
copyright: true
date: 2018-06-01 10:11:39
tags:
- Python
---

### 查看环境变量

```python
import os
print(os.sys.path)
```

### 添加环境变量

#### 临时添加，在一个shell窗口中

```python
import sys
sys.path.append(path)
```

但退出该shell窗口，即失效

#### 使用pth文件永久添加

使用pth文件，在 site-packages 文件中创建 .pth文件，将模块的路径写进去，一行一个路径，以下是一个示例，pth文件也可以使用注释：

```python
# .pth file for the  my project(这行是注释)
E:\DjangoWord
E:\DjangoWord\mysite
E:\DjangoWord\mysite\polls
```

这个不失为一个好的方法，但存在管理上的问题，而且不能在不同的python版本中共享 

#### 使用PYTHONPATH环境变量

使用PYTHONPATH环境变量，在这个环境变量中输入相关的路径，不同的路径之间用逗号（英文的！)分开，如果PYTHONPATH 变量还不存在，可以创建它！

路径会自动加入到sys.path中，而且可以在不同的python版本中共享，应该是一样较为方便的方法

### 参考

https://www.cnblogs.com/GODYCA/archive/2013/05/31/3110410.html

https://www.cnblogs.com/huchong/p/7349894.html