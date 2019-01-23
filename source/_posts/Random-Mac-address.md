---
title: 随机生成MAC地址的N种方法
date: 2018-12-27 11:43:34
categories:
tags:
- Linux
- Python
---

## 转载：[随机生成MAC地址的N种方法](http://www.361way.com/generate-random-mac-addresses/3244.html)

### Shell生成法

shell生成的方法是最多的的，同时也感觉也是最为简单高效的，这里列几种常用工具随机生成的方法：

#### openssl工具生成

```
yang@crunchbang:~$ openssl rand -hex 6 | sed 's/(..)/1:/g; s/.$//'a0:77:d4:ef:08:7dyang@crunchbang:~$ openssl rand 6 | xxd -p | sed 's/(..)/1:/g; s/:$//'3b:7f:95:c8:39:6d
```

<!--more-->

#### od生成

```
yang@crunchbang:~$ od -An -N10 -x  /dev/random  | md5sum | sed -r 's/^(.{10}).*$/1/; s/([0-9a-f]{2})/1:/g; s/:$//;'b0:85:1a:41:b1yang@crunchbang:~$yang@crunchbang:~$ od /dev/urandom -w6 -tx1 -An|sed -e 's/ //' -e 's/ /:/g'|head -n 1d8:d3:67:20:c5:f2
```



#### for循环生成

```
yang@crunchbang:~$ for i in {1..6}; do printf "%0.2X:" $[ $RANDOM % 0x100 ]; done | sed 's/:$/n/'8E:9E:FB:AE:FF:D2yang@crunchbang:~$ h=0123456789ABCDEF;for c in {1..12};do echo -n ${h:$(($RANDOM%16)):1};if [[ $((c%2)) = 0 && $c != 12 ]];then echo -n :;fi;done;echo19:7F:A9:41:E2:20
```

这里再次感叹下，语言本身没有高级贵贱，不要轻视了shell，shell能实现的干吗非得要用perl、python、php等去实现。



### perl生成法

```
yang@crunchbang:~$ perl -e 'printf("%.2x:",rand(255))for(1..5);printf("%.2xn",rand(255))'f8:42:c1:d4:a8:28yang@crunchbang:~$ perl -e 'print join(":", map { sprintf "%0.2X",rand(256) }(1..6))."n"'A7:02:BD:BC:59:E2
```

perl的强大与简洁无可争辩 。



### ruby生成法

```
yang@crunchbang:~$ ruby -e 'puts (1..6).map{"%0.2X"%rand(256)}.join(":")'CD:97:ED:52:B7:F4
```

这里使用的方法几乎和perl中的方法一样。



### python生成法

```
yang@crunchbang:~$ python -c "from itertools import imap; from random import randint; print ':'.join(['%02x'%x for x in imap(lambda x:randint(0,255), range(6))])"52:75:80:68:3a:cc
```

centos和redhat官方站点上也给出了一个python脚本：

```
#!/usr/bin/python
# macgen.py script to generate a MAC address for Red Hat Virtualization guests
#
import random
#
def randomMAC():
	mac = [ 0x00, 0x16, 0x3e,
		random.randint(0x00, 0x7f),
		random.randint(0x00, 0xff),
		random.randint(0x00, 0xff) ]
	return ':'.join(map(lambda x: "%02x" % x, mac))
#
print randomMAC()
```

在有virtinst.util模块时，也可以使用下面的简单语句生成新的mac和uuid：

```
#!/usr/bin/env python
#  -*- mode: python; -*-
print ""
print "New UUID:"
import virtinst.util ; print virtinst.util.uuidToString(virtinst.util.randomUUID())
print "New MAC:"
import virtinst.util ; print virtinst.util.randomMAC()
print ""
```

