---
title: Git中.gitignore文件的使用
copyright: true
date: 2018-08-03 10:48:55
tags:
- Git
---

## gitignore工作原理

当你对文件进行源代码管理时，`gitignore`插件会去查找当前编辑文件的目录或者其祖先目录中是否存在`.gitignore`配置文件。 如果存在，则`.gitignore`配置文件中对应配置的规则会生效，忽略对应的文件

<!--more-->

## 如何使用gitignore

gitignore使用起来非常的方便

- 在项目根目录创建一个`.gitignore`的文件
- 在这个文件中定义需要忽略的文件
- 给IDE安装gitignore插件
- [.gitignore文件模板](https://github.com/github/gitignore)
- [官方文档](https://git-scm.com/docs/gitignore)

## gitignore配置语法

- 以斜杠`/`开头表示目录
- 以星号`*`通配多个字符
- 以问号`?`通配单个字符
- 以方括号`[]`包含单个字符的匹配列表
- 以叹号`!`表示不忽略(跟踪)匹配到的文件或目录
- 以二个星号`**`开头表示匹配所有目录
- 以二个星号`/**`结尾表示匹配目录里的所有
- 以斜杠中间的二个星号`/**/`表示0或多个目录

## 示例

1. 规则：`fd1/*`

   说明：忽略目录 `fd1` 下的全部内容；注意，不管是根目录下的 `/fd1/` 目录，还是某个子目录 `/child/fd1/` 目录，都会被忽略；

2. 规则：`/fd1/*` 　　　　
   说明：忽略根目录下的 `/fd1/` 目录的全部内容；

3. 规则：

   ```
    /*
    !.gitignore
    !/foo
    !/foo/bar    
   ```

   说明：忽略全部内容，但是不忽略 `.gitignore` 文件、根目录下的 `/foo` 和 `/foo/bar `目录

4. 忽略指定文件/目录

   ```
   # 忽略指定文件
   HelloWrold.class
   
   # 忽略指定文件夹
   bin/
   bin/gen/
   ```

5. 通配符忽略规则

   通配符规则如下：

   ```
   # 忽略.class的所有文件
   *.class
   
   # 忽略名称中末尾为ignore的文件夹
   *ignore/
   
   # 忽略名称中间包含ignore的文件夹
   *ignore*/
   ```

## 查看gitignore规则

   如果你发下`.gitignore`写得有问题，需要找出来到底哪个规则写错了，可以用`git check-ignore`命令检查：

   ```
   $ git check-ignore -v HelloWorld.class
   .gitignore:1:*.class    HelloWorld.class
   ```

   可以看到`HelloWorld.class`匹配到了我们的第一条`*.class`的忽略规则所以文件被忽略了。

## 使用技巧

要忽略已存在的文件或路径时，如果此时代码仓库已存在该路径或文件，需要先移除本地的路径或文件，然后执行如下代码，最后再push到仓库：

- 如果要移除路径 dir

```
git rm --cached -r dir/
```

- 如果要移除文件 doc.txt

```
git rm --cached -r doc.txt
```

 ## 参考

[Git忽略文件.gitignore的使用](https://www.jianshu.com/p/a09a9b40ad20)

[代码管理工具Git的使用--gitignore](https://juejin.im/post/5b4a4cd5e51d4519520e5775)

[gitignore使用小记](gitignore使用小记)    

​    

​    

​    

​    

​    

​    

​    

​    