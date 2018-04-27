---
title: Linux命令小结
copyright: true
date: 2018-04-27 15:19:07
tags:
- Linux
---

### Linux下统计当前文件夹下的文件个数、目录个数

- **统计当前文件夹下文件的个数**

  ```shell
  ls -l |grep "^-"|wc -l
  ```

- **统计当前文件夹下目录的个数**

  ```shell
  ls -l |grep "^d"|wc -l
  ```

- **统计当前文件夹下文件的个数，包括子文件夹里的**

  ```shell
  ls -lR|grep "^-"|wc -l
  ```

- **统计文件夹下目录的个数，包括子文件夹里的**

  ```shell
  ls -lR|grep "^d"|wc -l
  ```

  ​

### Linux shell 提取文件名和目录名

**用于字符串的读取，提取和替换功能，可以使用${} 提取字符串**

- 提取文件名

  ```shell
  [root@localhost log]# var=/dir1/dir2/file.txt
  [root@localhost log]# echo ${var##*/}
  file.txt
  ```

- 提取后缀

  ```shell
  [root@localhost log]# echo ${var##*.}
  txt
  ```

- 提取不带后缀的文件名，分两步

  ```shell
  [root@localhost log]# tmp=${var##*/}
  [root@localhost log]# echo $tmp
  file.txt
  [root@localhost log]# echo ${tmp%.*}
  file
  ```

- 提取目录

  ```shell
  [root@localhost log]# echo ${var%/*}
  /dir1/dir2
  ```

**使用文件目录的专有命令basename和dirname**

- 提取文件名，注意：basename是一个命令，使用$(), 而不是${}

  ```shell
  [root@localhost log]# echo $(basename $var)
  file.txt
  ```

- 提取不带后缀的文件名

  ```shell
  [root@localhost log]# echo $(basename $var .txt)
  file
  ```

- 提取目录

  ```shell
  [root@localhost log]# dirname $var
  /dir1/dir2
  [root@localhost log]# echo $(dirname $var)
  /dir1/dir2
  ```

  ### [shell sort命令](http://www.cnblogs.com/kimbo/p/7263344.html)

  用法：sort [选项]... [文件]...
  串联排序所有指定文件并将结果写到标准输出。

  排序选项：

  >  -b, --ignore-leading-blanks	忽略前导的空白区域
  >
  >  -d, --dictionary-order	只考虑空白区域和字母字符
  >
  >  -f, --ignore-case	忽略字母大小写
  >
  >  -g, --general-numeric-sort	按照常规数值排序
  >
  >  -i, --ignore-nonprinting	只排序可打印字符
  >
  >  -n, --numeric-sort	根据字符串数值比较
  >
  >  -r, --reverse	逆序输出排序结果，降序排序
  >
  >  其他选项：
  >
  >  -c, --check, --check=diagnose-first	检查输入是否已排序，若已有序则不进行操作
  >  -k, --key=位置1[,位置2]	在位置1 开始一个key，在位置2 终止(默认为行尾)
  >
  >  -m, --merge	合并已排序的文件，不再进行排序
  >
  >  -o, --output=文件	将结果写入到文件而非标准输出
  >
  >  -t, --field-separator=分隔符	使用指定的分隔符代替非空格到空格的转换
  >
  >  -u, --unique	配合-c，严格校验排序；不配合-c，则只输出一次排序结果，默认安装ASCII码升序排列