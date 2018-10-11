---
title: Python调用linux的命令
date: 2018-09-26 14:09:30
categories:
- Python
tags:
- Python
---

## 转载：[python调用linux的命令](https://www.cnblogs.com/hujq1029/p/7096247.html)

有时候难免需要直接调用Shell命令来完成一些比较简单的操作，比如mount一个文件系统之类的。那么我们使用Python如何调用Linux的Shell命令？下面来介绍几种常用的方法：

<!--more-->

## os模块

### os模块的exec方法族

[python](http://lib.csdn.net/base/python)的exec系统方法同Unix的exec系统调用是一致的。这些方法适用于在子进程中调用外部程序的情况，因为外部程序会替换当前进程的代码，不会返回。( 这个看了点 help(os)  --> search "exec" 的相关介绍，但是没太搞明白咋使用)



### os模块的system方法

system方法会创建子进程运行外部程序，方法只返回外部程序的运行结果。这个方法比较适用于外部程序没有输出结果的情况。 

```
>>> import os  
>>> os.system("echo \"Hello World\"")   # 直接使用os.system调用一个echo命令  
Hello World         ——————> 打印命令结果  
0                   ——————> What's this ? 返回值？  
>>> val = os.system("ls -al | grep \"log\" ")   # 使用val接收返回值  
-rw-r--r--  1 root       root       6030829 Dec 31 15:14 log    ——————> 此时只打印了命令结果  
>>> print val             
0                   ——————> 注意，此时命令正常运行时，返回值是0  
>>> val = os.system("ls -al | grep \"log1\" ")  
>>> print val         
256                 ——————> 使用os.system调用一个没有返回结果的命令，返回值为256～  
>>>   
```


注意：上面说了，此方法只返回外部程序的结果，也就是os.system的结果，所以如果你想接收命令的返回值，接着向下看～



### os模块的popen方法

当需要得到外部程序的输出结果时，本方法非常有用，返回一个类文件对象，调用该对象的read()或readlines()方法可以读取输出内容。比如使用urllib调用Web API时，需要对得到的数据进行处理。os.popen(cmd) 要得到命令的输出内容，只需再调用下read()或readlines()等 如a=os.popen(cmd).read()

```
>>> os.popen('ls -lt')                  # 调用os.popen（cmd）并不能得到我们想要的结果  
<open file 'ls -lt ', mode 'r' at 0xb7585ee8>  
>>> print os.popen('ls -lt').read()     # 调用read()方法可以得到命令的结果  
total 6064  
-rwxr-xr-x 1 long       long            23 Jan  5 21:00 hello.sh  
-rw-r--r-- 1 long       long           147 Jan  5 20:26 Makefile  
drwxr-xr-x 3 long       long          4096 Jan  2 19:37 test  
-rw-r--r-- 1 root       root       6030829 Dec 31 15:14 log  
drwxr-xr-x 2 long       long          4096 Dec 28 09:36 pip_build_long  
drwx------ 2 Debian-gdm Debian-gdm    4096 Dec 23 19:08 pulse-gylJ5EL24GU9  
drwx------ 2 long       long          4096 Jan  1  1970 orbit-long  
>>> val = os.popen('ls -lt').read()     # 使用变量可以接收命令返回值  
>>> if "log" in val:                    # 我们可以使用in来判断返回值中有木有一个字符串  
...     print "Haha,there is the log"  
... else:  
...     print "No,not happy"  
...  
Haha,there is the log  
```



## commands 模块

使用commands模块的getoutput方法，这种方法同popend的区别在于popen返回的是一个类文件对象，而本方法将外部程序的输出结果当作字符串返回，很多情况下用起来要更方便些。
主要方法:  

\*   commands.getstatusoutput(cmd)         返回(status, output)
\*   commands.getoutput(cmd)                   只返回输出结果
\*   commands.getstatus(file)                     返回ls -ld file的执行结果字符串，调用了getoutput，不建议使用此方法

```
long@zhouyl:/tmp/tests$ python  
Python 2.7.3 (default, Jan  2 2013, 16:53:07)   
[GCC 4.7.2] on linux2  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import commands  
>>> commands.getstatusoutput('ls -lt')      # 返回(status, output)  
(0, 'total 5900\n-rwxr-xr-x 1 long long      23 Jan  5 21:34 hello.sh\n-rw-r--r-- 1 long long     147 Jan  5 21:34 Makefile\n-rw-r--r-- 1 long long 6030829 Jan  5 21:34 log')  
>>> commands.getoutput('ls -lt')            # 返回命令的输出结果（貌似和Shell命令的输出格式不同哈～）  
'total 5900\n-rwxr-xr-x 1 long long      23 Jan  5 21:34 hello.sh\n-rw-r--r-- 1 long long     147 Jan  5 21:34 Makefile\n-rw-r--r-- 1 long long 6030829 Jan  5 21:34 log'  
>>> commands.getstatus('log')               # 调用commands.getoutput中的命令对'log'文件进行相同的操作  
'-rw-r--r-- 1 long long 6030829 Jan  5 21:34 log'  
>>>   
```



## subprocess模块


根据Python官方文档说明，subprocess模块用于取代上面这些模块。有一个用Python实现的并行ssh工具—mssh，代码很简短，不过很有意思，它在线程中调用subprocess启动子进程来干活。

```
>>> from subprocess import call  
>>> call(["ls", "-l"]) 
```

subprocess与system相比的优势是它更灵活（你可以得到标准输出，标准错误，“真正”的状态代码，更好的错误处理，等..）。我认为使用os.system已过时，或即将过时。



## 众方法的比较以及总结

### 关于 os.system

os.system("some_command with args")将命令以及参数传递给你的系统shell，这很好，因为你可以用这种方法同时运行多个命令并且可以设置管道以及输入输出重定向。比如：
os.system("some_command < input_file | another_command > output_file")
然而，虽然这很方便，但是你需要手动处理shell字符的转义，比如空格等。此外，这也只能让你运行简单的shell命令而且不能运行外部程序。



### 关于os.popen

使用stream = os.popen("some_command with args")也能做与os.system一样的事，与os.system不同的是os.popen会返回一个类文件对象，使用它来访问标准输入、输出。



### 关于subprocess.popen

subprocess模块的Popen类，意图作为os.popen的替代，但是因为其很全面所以比os.popen要显得稍微复杂。
比如你可以使用  print Popen("echo Hello World", stdout=PIPE, shell=True).stdout.read()  来替代  print os.popen("echo Hello World").read()。但是相比之下它使用一个统一的类包括4中不同的popen函数还是不错的。



### 关于subprocess.call

subprocess模块的call函数。它基本上就像Popen类并都使用相同的参数，但是它只简单的等待命令完成并给你返回代码。比如：
return_code = subprocess.call("echo Hello World", shell=True)
os模块中还有C中那样的fork/exec/spawn函数，但是我不建议直接使用它们。subprocess可能更加适合你。

## 参考
[1] http://demi-panda.com/2013/01/25/python-shell-command/index.html
[2] http://m.blog.csdn.net/blog/overstack/9295995
[3] http://blog.csdn.net/swiftshow/article/details/7755543

下面是对于文中所涉及的内容的python官方文档：

[4] http://docs.python.org/library/subprocess.html#replacing-older-functions-with-the-subprocess-module       -- 关于使用subprocess 替代老的方法

[5] http://docs.python.org/lib/os-process.html     -- os的exec方法族以及system方法

[6] http://docs.python.org/lib/os-newstreams.html    -- os的popen方法

[7] http://docs.python.org/lib/node528.html    -- os的subprocess介绍