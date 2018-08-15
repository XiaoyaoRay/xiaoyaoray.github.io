---
title: 在Mac OS X系统下 用dd命令将iso镜像写入u盘
copyright: true
categories:
- MacOS
tags:
- MacOS
---


### 操作概述

1、找出U盘挂载的路径，使用如下命令：diskutil list
2、将U盘unmount（将N替换为挂载路径）：diskutil unmountDisk /dev/disk[N]
3、写入U盘：sudo dd if=iso路径 of=/dev/rdisk[N] bs=1m  rdisk 中加入r可以让写入速度加快
<!--more-->
### 具体操作步骤

1. 查看u盘盘符

```Shell
lapommedeMacBook-Pro:~ lapomme$ diskutil list
/dev/disk0 (internal, physical):
#: TYPE NAME SIZE IDENTIFIER
0: GUID_partition_scheme *251.0 GB disk0
1: EFI EFI 209.7 MB disk0s1
2: Apple_CoreStorage Macintosh HD 250.1 GB disk0s2
3: Apple_Boot Recovery HD 650.0 MB disk0s3
/dev/disk1 (internal, virtual):
#: TYPE NAME SIZE IDENTIFIER
0: Apple_HFS Macintosh HD +249.8 GB disk1
Logical Volume on disk0s2
E2BD4617-5A22-46A9-A6F4-D54E3EE92BBC
Unencrypted
/dev/disk2 (external, physical):
#: TYPE NAME SIZE IDENTIFIER
0: FDisk_partition_scheme *62.0 GB disk2
1: DOS_FAT_32 UNTITLED 1 62.0 GB disk2s1
/dev/disk3 (disk image):
#: TYPE NAME SIZE IDENTIFIER
0: FDisk_partition_scheme +3.1 GB disk3
1: 0x17 3.0 GB disk3s1
2: DOS_FAT_12 NO NAME 110.1 MB disk3s2 
```



2. 取消挂载U盘

```
lapommedeMacBook-Pro:~ lapomme$ diskutil umountDisk /dev/disk2
Unmount of all volumes on disk2 was successful
```



3. 用dd命令写入U盘

说明：

（1）sudo dd if=源路径 of=/dev/r卷标 bs=1m ［‘r’ 会让命令执行加快一点］ ［‘bs’为一次填充的容量］

*（2）获取映像名称和完整路径可以直接将文件拖入终端，即在终端中显示*

```
lapommedeMacBook-Pro:~ lapomme$ sudo dd if=/xxx.iso of=/dev/rdisk2 bs=1m
2934+1 records in
2934+1 records out
3076767744 bytes transferred in 149.567568 secs (20571089 bytes/sec)
```



4. 查看磁盘进度，可以用iostat命令查看磁盘写入状态

``` shell
lapommedeMacBook-Pro:~ lapomme$ iostat -w 2
disk0 disk2 disk3 cpu load average
KB/t tps MB/s KB/t tps MB/s KB/t tps MB/s us sy id 1m 5m 15m
102.71 25 2.49 598.70 0 0.15 24.76 1 0.02 5 4 90 2.04 1.71 1.69
512.00 48 23.93 1024.00 24 23.93 0.00 0 0.00 1 3 96 2.11 1.74 1.69
473.00 26 11.99 1024.00 12 11.98 0.00 0 0.00 3 3 93 2.11 1.74 1.69
491.68 50 23.99 1024.00 24 23.99 0.00 0 0.00 24 8 68 2.11 1.74 1.69
```

 

5. 操作完毕后将U盘弹出

```
lapommedeMacBook-Pro:~ lapomme$ diskutil eject /dev/disk2
Disk /dev/disk2 ejected
```

### 附录

将iso转换成dmg 转iso (刻录非必须)

```
UDRW 替换为 UDTO
```

```
lapommedeMacBook-Pro:~ lapomme$ sudo hdiutil convert -format UDRW -o /linux.dmg kali.iso
Password:
正在读取Master Boot Record（MBR：0）…
正在读取Kali Live （Apple_ISO：1）…
正在读取（Windows_NTFS_Hidden：2）…
............................................................................
正在读取（DOS_FAT_12：3）…
..............................................................................
已耗时：10.178s
速度：288.3M 字节/秒
节省：0.0%
created: /linux.dmg
```

## 参考

https://www.cnblogs.com/bindong/p/5818497.html

http://osxdaily.com/2015/06/05/copy-iso-to-usb-drive-mac-os-x-command/
