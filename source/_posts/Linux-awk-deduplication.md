---
title: awk去重
copyright: true
date: 2018-05-24 09:43:05
categories:
- Linux
tags:
- Linux
- awk
---
#### 原文

```
[root@test--0007 ~]# cat test.txt
abc 13
abc 3
a d a
a d b
a a a
c c c
a a a
a d a
35 13 1
[root@test--0007 ~]#
```
<!--more-->
#### 去重显示没有重复的行

```
[root@test--0007 ~]# cat test.txt |awk '!a[$1]++{print}'
abc 13
a d a
c c c
35 13 1
[root@test--0007 ~]#
```

#### 去除重复的行

```
[root@test--0007 ~]# cat test.txt |awk '!a[$0]++{print}'
abc 13
abc 3
a d a
a d b
a a a
c c c
35 13 1
[root@test--0007 ~]#
```

#### 只显示重复的行

```
[root@test--0007 ~]# cat test.txt |awk 'a[$0]++{print}'
a a a
a d a
[root@test--0007 ~]#
```

#### 参考

https://www.cnblogs.com/chongchong88/p/6085905.html
