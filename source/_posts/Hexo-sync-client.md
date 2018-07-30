---
title: Hexo在不同终端同步
copyright: true
date: 2018-04-13 14:47:34
tags:
- Hexo
---

### 在hexo的目录下初始化github

```
git init  //初始化本地仓库
```
<!--more-->
### 在`.gitignore`文件中添加忽略的文件和文件夹

```
cat .gitignore
node_modules
debug.log
db.json
package-lock.json
public
```

### push到GitHub

```
git add .
git commit -m "Blog Source Hexo"
git branch hexo  //新建hexo分支
git checkout hexo  //切换到hexo分支上
git remote add origin git@github.com:yourname/yourname.github.io.git  //将本地与Github项目对接
```



###其他终端clone，安装npm的modules，生成blog

```
git clone -b hexo git@github.com:yourname/yourname.github.io.git //将Github中hexo分支clone到本地
cd  yourname.github.io  //切换到刚刚clone的文件夹内
npm install    //注意，这里一定要切换到刚刚clone的文件夹内执行，安装必要的所需组件，不用再init
hexo d -g   //push更新完分支之后将自己写的博客对接到自己搭的博客网站上，同时同步了Github中的master
```

### 参考

[如何解决github+Hexo的博客多终端同步问题](https://blog.csdn.net/Monkey_LZL/article/details/60870891)
