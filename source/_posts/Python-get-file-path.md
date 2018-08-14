---
title: python中__file__ 与argv[0]使用
copyright: true
date: 2018-08-14 10:53:04
tags:
- Python
---

在python下，获取当前执行主脚本的方法有两个：sys.argv[0]和\_\_file\_\_

### sys.argv[0]

获取主执行文件路径的最佳方法是用sys.argv[0]，它可能是一个相对路径，所以再取一下abspath是保险的做法，像这样：

```
import os,sys
dirname, filename = os.path.split(os.path.abspath(sys.argv[0]))
print "running from", dirname
print "file is", filename
```

<!--more-->

执行结果

```
running from /root
file is test.py 
```

#### sys.path[0]，os.getcwd()，sys.argv[0]

```
print "3",sys.path[0] 
print "4",sys.argv[0]
print "5",os.getcwd()
```

执行结果

```
3 /devcode/user/bin/dw_clsfd #运行脚本目录
4 /dw/etl/home/dev/bin/dw_clsfd/test.py #当前脚本的目录
5 /devcode/user/bin/dw_clsfd #运行脚本目录
```

**通过sys.path[0],os.getcwd()获得的是执行脚本的目录**

### \_\_file\_\_

\__file__ 是用来获得模块所在的路径的，这可能得到的是一个相对路径，比如在脚本test.py中写入：

\#!/usr/bin/env python
print \__file__

- 按相对路径./test.py来执行，则打印得到的是相对路径，
- 按绝对路径执行则得到的是绝对路径。
- 而按用户目录来执行（\~/practice/test.py），则得到的也是绝对路径（~被展开）
- 所以为了得到绝对路径，我们需要 os.path.realpath(\__file__)。

而在Python控制台下，直接使用print \_\_file\_\_是会导致  name ‘\__file\_\_’ is not defined错误的，因为这时没有在任何一个脚本下执行，自然没有 \_\_file\_\_的定义了。

#### python中os.path.dirname(\__file__)

```
print "7",os.path.dirname(__file__)

(1).当"print os.path.dirname(__file__)"所在脚本是以完整路径被运行的， 那么将输出该脚本所在的完整路径，比如：
python /dw/etl/home/dev/bin/dw_clsfd/test.py 
7 /dw/etl/home/dev/bin/dw_clsfd #当前脚本所在的目录

(2).当"print os.path.dirname(__file__)"所在脚本是以相对路径被运行的， 那么将输出空目录，比如：
python test.py
那么将输出空字符串
```

#### 其他

```
print "6",__file__
print "7",os.path.realpath(__file__)
print "8",os.path.abspath(__file__)
print "9",os.path.dirname(os.path.realpath(__file__))
print "10",os.path.dirname(os.path.abspath(__file__))
```

执行结果

```
6 /dw/etl/home/dev/bin/dw_clsfd/test.py
7 /devcode/user/bin/dw_clsfd/test.py #脚本运行目录
8 /dw/etl/home/dev/bin/dw_clsfd/test.py
9 /devcode/user/bin/dw_clsfd  #脚本运行目录
10 /dw/etl/home/dev/bin/dw_clsfd
```

 

### \__file__和argv[0]差异

在主执行文件中时，两者没什么差异，不过要是在不同的文件下，就不同了，下面示例：

```
C:\junk\so&gt;type \junk\so\scriptpath\script1.py
import sys, os
print "script: sys.argv[0] is", repr(sys.argv[0])
print "script: __file__ is", repr(__file__)
print "script: cwd is", repr(os.getcwd())
import whereutils
whereutils.show_where()
 
C:\junk\so&gt;type \python26\lib\site-packages\whereutils.py
import sys, os
def show_where():
    print "show_where: sys.argv[0] is", repr(sys.argv[0])
    print "show_where: __file__ is", repr(__file__)
    print "show_where: cwd is", repr(os.getcwd())
 
C:\junk\so&gt;\python26\python scriptpath\script1.py
script: sys.argv[0] is 'scriptpath\\script1.py'
script: __file__ is 'scriptpath\\script1.py'
script: cwd is 'C:\\junk\\so'
show_where: sys.argv[0] is 'scriptpath\\script1.py'
show_where: __file__ is 'C:\\python26\\lib\\site-packages\\whereutils.pyc'
show_where: cwd is 'C:\\junk\\so'
```

**所以一般来说，argv[0]要更可靠些**

### 参考

[python \__file__ 与argv[0]](https://blog.csdn.net/weixin_37746272/article/details/78980259)

[python解惑之 \__file__ 与argv[0]](https://www.cnblogs.com/rrxc/p/3973136.html)