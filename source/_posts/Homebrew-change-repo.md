---
title: 更换Homebrew的更新源
copyright: true
date: 2018-07-26 16:54:31
categories:
- MacOS
tags:
- MacOS
---

**欢迎参与讨论，转载请注明出处。** 
　　**本文转载自https://musoucrow.github.io/2017/03/29/brew_changing/**

## 前言

　　更换Homebrew的更新源的教程，在网上数不胜数，然内容大多大同小异且述之不详，且未提及版本上的差异。故作此文，以正视听。 
　　在阅读此文之前，你需要了解[Homebrew](https://brew.sh/)和[Git](https://git-scm.com/)并安装了它们。并且对于Homebrew官方更新源的速度赶到不满且不打算利用其它手段解决（如VPN），或者看了其它文章感到不求甚解，那么此文对你而言是有价值的。
<!--more-->
## 更新源的机制

　　Homebrew的更新源由三部分组成：本体（brew.git）、核心（homebrew-core.git）以及二进制预编译包（homebrew-bottles）。 
　　在很多教程中，只会提及到更换本体，而未涉及到核心与二进制预编译包的更换。这样实际上效果是不完全的（尽管这样也无法做到完全，毕竟有一些软件包的地址是不被收录的，只能从它们提供的链接处下载）。 
　　从.git的后缀名可以看出，Homebrew的更新源是以Git仓库的形式存在的，这便是为什么需要用到Git的原因。也正是如此，使得可以对其进行克隆，成为新源。

## 更新源的选择

　　默认官方的更新源都是存放在[GitHub](https://github.com/)上的，这也是中国大陆用户访问缓慢的原因，一般来说我们会更倾向选择国内提供的更新源，在此推荐[中国科大](https://mirrors.ustc.edu.cn/)以及[清华大学](https://mirrors.tuna.tsinghua.edu.cn/)提供的更新源，因为它们能够完整以上源组成的三个部分。并且在此感谢他们为大家提供的服务。 

## 替换更新源

```
# 替换brew.git:
$ cd "$(brew --repo)"
# 中国科大:
$ git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
# 清华大学:
$ git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

# 替换homebrew-core.git:
$ cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
# 中国科大:
$ git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
# 清华大学:
$ git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

# 替换homebrew-bottles:
# 中国科大:
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
$ source ~/.bash_profile
# 清华大学:
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
$ source ~/.bash_profile

# 应用生效:
$ brew update
```

　　以上在中国科大和清华大学任选其一即可，在使用其他源的时候，最好先尝试访问其链接看看是否健在，并且因为历史原因，最初的brew.git是叫homebrew.git的，而现在部分更新源早已随官方更名，所以切记要验证。 
　　并且没有严格规定必须三个组成部分必须是来自同一提供，可随性发挥。 
　　且Homebrew在早期版本中更新源的是在/usr/local目录下的，而现在是在/usr/local/Homebrew，不过应该都是可以使用`"$(brew --repo)"`来自动指向目录的，所以无需理会。 
　　如果你之前折腾过不少导致你的Homebrew有点问题，那么可以尝试使用如下方案：

```
# 诊断Homebrew的问题:
$ brew doctor

# 重置brew.git设置:
$ cd "$(brew --repo)"
$ git fetch
$ git reset --hard origin/master

# homebrew-core.git同理:
$ cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
$ git fetch
$ git reset --hard origin/master

# 应用生效:
$ brew update
```

## 重置更新源

　　所谓有进则有退，在某些时候也有换回官方源的需求。

```
# 重置brew.git:
$ cd "$(brew --repo)"
$ git remote set-url origin https://github.com/Homebrew/brew.git

# 重置homebrew-core.git:
$ cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
$ git remote set-url origin https://github.com/Homebrew/homebrew-core.git
```

　　至于homebrew-bottles，本质上作为一个环境变量的存在，之前的命令也只是将其写入到`/usr/.bash_profile`中，并且只是在文件尾部添加一行。所以之前的命令不推荐重复执行，在未掌握相关命令技巧的前提下，我推荐直接去修改`.bash_profile`文件：![bash_profile](https://musoucrow.github.io/images/brew_changing/bash_profile.png)
　　当然这里的主题是重置更新源，所以我们直接选择删除环境变量HOMEBREW_BOTTLE_DOMAIN，使其成为默认值即可。 
　　当然，最后不要忘记`$ brew update`进行应用。

## 后记

　　在完成更新源的更换后，我们可以使用`$ brew upgrade`将现有的软件进行更新至最新版本，这样便能很直接的看出速度上的变化了。最后不要忘记`$ brew cleanup`将旧有的软件安装包进行清理。
