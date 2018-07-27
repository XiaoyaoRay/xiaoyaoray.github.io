---
title: Python中os.path模块常用用法
copyright: true
date: 2018-06-01 10:26:55
tags:
- Python
---

### 转载： [Python中os.path模块下常用的用法总结](https://www.cnblogs.com/renpingsheng/p/7065565.html)

### abspath

```
返回一个目录的绝对路径
Return an absolute path.
```
<!--more-->
```
>>> os.path.abspath("/etc/sysconfig/selinux")
'/etc/sysconfig/selinux'
>>> os.getcwd()
'/root'
>>> os.path.abspath("python_modu")
'/root/python_modu'
```

### basename

```
返回一个目录的基名
Returns the final component of a pathname
```

```
>>> os.path.basename("/etc/sysconfig/selinux")
'selinux'
>>> os.path.basename("/usr/local/python3/bin/python3")
'python3'
```

### dirname

```
返回一个目录的目录名
Returns the directory component of a pathname
```

```
>>> os.path.dirname("/etc/sysconfig/selinux")
'/etc/sysconfig'
>>> os.path.dirname("/usr/local/python3/bin/python3")
'/usr/local/python3/bin'
```

### exists

```
测试指定文件是否存在
Test whether a path exists.  Returns False for broken symbolic links
```

```
>>> os.path.exists("/home/egon")
False
>>> os.path.exists("/root")
True
>>> os.path.exists("/usr/bin/python")
True
```

### getatime

```
得到指定文件最后一次的访问时间
Return the last access time of a file, reported by os.stat().
```

```
>>> os.stat("/root/test.sh")
os.stat_result(st_mode=33261, st_ino=100684935, st_dev=2050, st_nlink=1, st_uid=0, st_gid=0, st_size=568, st_atime=1498117664, st_mtime=1496629059, st_ctime=1498117696)
>>> os.path.getatime("/root/test.sh")
1498117664.2808378
```

### getctime

```
得到指定文件最后一次的改变时间
Return the metadata change time of a file, reported by os.stat().
```

```
>>> os.stat("/root/test.sh")
os.stat_result(st_mode=33261, st_ino=100684935, st_dev=2050, st_nlink=1, st_uid=0, st_gid=0, st_size=568, st_atime=1498117664, st_mtime=1496629059, st_ctime=1498117696)
>>> os.path.getctime("/root/test.sh")
1498117696.039542
```

### getmtime

```
得到指定文件最后一次的修改时间
Return the last modification time of a file, reported by os.stat().
```

```
>>> os.stat("/root/test.sh")
os.stat_result(st_mode=33261, st_ino=100684935, st_dev=2050, st_nlink=1, st_uid=0, st_gid=0, st_size=568, st_atime=1498117664, st_mtime=1496629059, st_ctime=1498117696)
>>> os.path.getmtime("/root/test.sh")
1496629059.9313989
```

### getsize

```
得到得到文件的大小
Return the size of a file, reported by os.stat().
```

```
>>> os.stat("/root/test.sh")
os.stat_result(st_mode=33261, st_ino=100684935, st_dev=2050, st_nlink=1, st_uid=0, st_gid=0, st_size=568, st_atime=1498117664, st_mtime=1496629059, st_ctime=1498117696)
>>> os.path.getsize("/root/test.sh")
568
```

### isabs

```
测试参数是否是绝对路径
Test whether a path is absolute
```

```
>>> os.path.isabs("python_modu")
False
>>> os.path.isabs("/etc/sysconfig")
True
```

### isdir

```
测试指定参数是否是目录名
Return true if the pathname refers to an existing directory.
```

```
>>> os.path.isdir("/etc/sysconfig/selinux")
False
>>> os.path.isdir("/home")
True
```

### isfile

```
测试指定参数是否是一个文件
Test whether a path is a regular file
```

```
>>> os.path.isfile("/home")
False
>>> os.path.isfile("/etc/sysconfig/selinux")
True
```

### islink

```
测试指定参数是否是一个软链接
Test whether a path is a symbolic link
```

```
>>> os.path.islink("/etc/sysconfig/selinux")
True
>>> os.path.islink("/etc/sysconfig/nfs")
False
```

### ismount

```
测试指定参数是否是挂载点
Test whether a path is a mount point
```

```
>>> os.path.ismount("/mnt/cdrom")
False
以上是未挂载光盘，现在把光盘挂载到/mnt/cdrom下，再进行测试
>>> os.path.ismount("/mnt/cdrom")
True
```

### join

```
join(a, *p)
将目录名和文件的基名拼接成一个完整的路径
Join two or more pathname components, inserting '/' as needed.
If any component is an absolute path, all previous path components
will be discarded.  An empty last part will result in a path that
ends with a separator.
```

```
>>> for filename in os.listdir("/home"):
...     print(os.path.join("/tmp",filename))
... 
/tmp/a
/tmp/f1.txt
```

### realpath

```
返回指定文件的标准路径，而非软链接所在的路径
Return the canonical path of the specified filename, eliminating any
symbolic links encountered in the path.
```

```
>>> os.path.realpath("/etc/sysconfig/selinux")
'/etc/selinux/config'
>>> os.path.realpath("/usr/bin/python")
'/usr/bin/python2.7'
```

### samefile

```
测试两个路径是否指向同一个文件
Test whether two pathnames reference the same actual file
```

### sameopenfile

```
测试两个打开的文件是否指向同一个文件
Test whether two open file objects reference the same file
```

### split

```
分割目录名，返回由其目录名和基名给成的元组
Split a pathname.  Returns tuple "(head, tail)" where "tail" is
everything after the final slash.  Either part may be empty.
```

```
>>> os.path.split("/tmp/f1.txt")
('/tmp', 'f1.txt')
>>> os.path.split("/home/test.sh")
('/home', 'test.sh')
```

### splitext

```
分割文件名，返回由文件名和扩展名组成的元组
Split the extension from a pathname.
Extension is everything from the last dot to the end, ignoring
leading dots.  Returns "(root, ext)"; ext may be empty.
>>> os.path.splitext("/home/test.sh")
('/home/test', '.sh')
>>> os.path.splitext("/tmp/f1.txt")
('/tmp/f1', '.txt')
```
