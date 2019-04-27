---
title: 获取当前脚本的绝对路径
date: 2019-02-11 15:46:35
categories:
- Shell
tags:
- Shell
---

## 转载：[bash shell:获取当前脚本的绝对路径(pwd/readlink)](https://blog.csdn.net/10km/article/details/51906821)

有时候，我们需要知道当前执行的输出shell脚本的所在绝对路径，可以用dirname实现。 
我们知道 dirname 可以获取一个文件所在的路径，dirname的用处是：

> 输出已经去除了尾部的”/”字符部分的名称；如果名称中不包含”/”， 
> 则显示”.”(表示当前目录)。

<!--more-->

下面是dirname的命令行说明： 

```
[root@deploy deploy]# dirname --help

Usage: dirname [OPTION] NAME...
Output each NAME with its last non-slash component and trailing slashes
removed; if NAME contains no /'s, output '.' (meaning the current directory).

  -z, --zero     separate output with NUL rather than newline
      --help     display this help and exit
      --version  output version information and exit

Examples:
  dirname /usr/bin/          -> "/usr"
  dirname dir1/str dir2/str  -> "dir1" followed by "dir2"
  dirname stdio.h            -> "."

GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
Report dirname translation bugs to <http://translationproject.org/team/>
For complete documentation, run: info coreutils 'dirname invocation'
```



从上面的描述可知道，直接从dirname返回的未必是绝对路径，取决于提供给dirname的参数是否是绝对路径。 
所以下面这样的代码中`SHELL_FOLDER`中不一定是绝对路径

```
SHELL_FOLDER=$(dirname "$0")
```


需要用cd和pwd命令配合获取脚本所在绝对路径，正确的写法是这样的

```
SHELL_FOLDER=$(cd "$(dirname "$0")";pwd)
```


如果你觉得上面的写法比较麻烦，还有一个方式获取脚本的绝对路径,就是借助readlink命令，下面是readlink的命令行说明： 

```
[root@deploy deploy]# readlink --help
Usage: readlink [OPTION]... FILE...
Print value of a symbolic link or canonical file name

  -f, --canonicalize            canonicalize by following every symlink in
                                every component of the given name recursively;
                                all but the last component must exist
  -e, --canonicalize-existing   canonicalize by following every symlink in
                                every component of the given name recursively,
                                all components must exist
  -m, --canonicalize-missing    canonicalize by following every symlink in
                                every component of the given name recursively,
                                without requirements on components existence
  -n, --no-newline              do not output the trailing delimiter
  -q, --quiet,
  -s, --silent                  suppress most error messages
  -v, --verbose                 report error messages
  -z, --zero                    separate output with NUL rather than newline
      --help     display this help and exit
      --version  output version information and exit

GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
Report readlink translation bugs to <http://translationproject.org/team/>
For complete documentation, run: info coreutils 'readlink invocation'
```




所以用readlink命令我们可以直接获取$0参数的全路径文件名，然后再用dirname获取其所在的绝对路径：

```
SHELL_FOLDER=$(dirname $(readlink -f "$0"))
```

