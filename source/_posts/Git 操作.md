---
title: Git 操作
copyright: true
tags: 
- Git
---


#### 删除远程分支

- 查看所有分支，找到要删除的分支名称

  ```shell
  git branch -a
  ```

- 运行命令删除远程分支

  ```shell
  git push origin --delete <branch name>
  ```


#### [子模块](https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)

- 通过`git submodule add`将外部项目加为子模块

  ```shell
  $ git submodule add git://github.com/chneukirchen/rack.git rack
  Initialized empty Git repository in /opt/subtest/rack/.git/
  remote: Counting objects: 3181, done.
  remote: Compressing objects: 100% (1534/1534), done.
  remote: Total 3181 (delta 1951), reused 2623 (delta 1603)
  Receiving objects: 100% (3181/3181), 675.42 KiB | 422 KiB/s, done.
  Resolving deltas: 100% (1951/1951), done.
  ```

- 克隆一个带子模块的项目

  这里你将克隆一个带子模块的项目。当你接收到这样一个项目，你将得到了包含子项目的目录，但里面没有文件：

  ```shell
  $ git clone git://github.com/schacon/myproject.git
  Initialized empty Git repository in /opt/myproject/.git/
  remote: Counting objects: 6, done.
  remote: Compressing objects: 100% (4/4), done.
  remote: Total 6 (delta 0), reused 0 (delta 0)
  Receiving objects: 100% (6/6), done.
  $ cd myproject
  $ ls -l
  total 8
  -rw-r--r--  1 schacon  admin   3 Apr  9 09:11 README
  drwxr-xr-x  2 schacon  admin  68 Apr  9 09:11 rack
  $ ls rack/
  ```

  `rack`目录存在了，但是是空的。你必须运行两个命令：`git submodule init`来初始化你的本地配置文件，`git submodule update`来从那个项目拉取所有数据并检出你上层项目里所列的合适的提交：

  ```shell
  $ git submodule init
  Submodule 'rack' (git://github.com/chneukirchen/rack.git) registered for path 'rack'
  $ git submodule update
  Initialized empty Git repository in /opt/myproject/rack/.git/
  remote: Counting objects: 3181, done.
  remote: Compressing objects: 100% (1534/1534), done.
  remote: Total 3181 (delta 1951), reused 2623 (delta 1603)
  Receiving objects: 100% (3181/3181), 675.42 KiB | 173 KiB/s, done.
  Resolving deltas: 100% (1951/1951), done.
  Submodule path 'rack': checked out '08d709f78b8c5b0fbeb7821e37fa53e69afcf433'
  ```

  ​