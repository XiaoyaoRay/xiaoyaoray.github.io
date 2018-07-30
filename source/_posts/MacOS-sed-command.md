---
title: MacOS-sed-command
copyright: true
date: 2018-07-30 18:00:16
tags:
- Mac
---

## Mac中使用sed修改文件出错解决方法

sed是linux命令，用于处理文件内容（修改，替换等），mac中都可以使用，但发现相同的替换命令在linux可以正常执行，在mac则执行失败

<!--more-->

### 出错原因

在linux执行正常，但在mac环境下执行出现以下错误：

```
# 命令
sed -i "s/shell/test/g" $(grep 'shell' -rl ./)
#错误信息
sed: 1: ".//make-RobotFramework- ...": invalid command code .
```

man sed 查看原因，找到 -i 参数的说明

> -i extension 
> Edit files in-place, saving backups with the specified extension. If a zero-length extension is given, no backup will be saved. It is not recommended to 
> give a zero-length extension when in-place editing files, as you risk corruption or partial content in situations where disk space is exhausted, etc.

原来sed -i需要带一个字符串作为备份源文件的文件名称，如果这个字符串长度为0，则不备份。 
例如执行

```
# 加_bak备份,则会创建一个xxx_bak的备份文件，文件内容为修改前的xxx内容
sed -i "_bak" "s/shell/test/g" $(grep 'shell' -rl ./)

# 空字符不备份
sed -i "" "s/shell/test/g" $(grep 'shell' -rl ./)
```



### 参考

[mac环境使用sed修改文件出错的解决方法](https://blog.csdn.net/fdipzone/article/details/51253955)

[sed: 1: “…”: invalid command code on Mac OS](https://blog.csdn.net/beijihukk/article/details/68947083)







