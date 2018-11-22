---
title: CentOS7 添加或删除用户
date: 2018-11-19 10:13:11
categories:
- Linux
tags:
- Linux
- CentOS
---

### 添加用户

- root用户，运行以下命令

  ```
  adduser <username>
  ```

- 非root用户，运行以下命令

  ```
  sudo adduser <username>
  ```



<!--more-->



#### 用户添加`sudo`权限

- root用户，运行以下命令

  ```
  gpasswd -a <username> wheel
  ```

- 非root用户，运行以下命令

  ```
  sudo gpasswd -a <username> wheel
  ```

- 用户添加好`sudo`权限后，命令需要用`root`权限时需要以下命令格式：

  ```
  sudo <some_command>
  ```

#### 查看`sudo`权限的用户

```
sudo lid -g wheel
```

### 删除用户

- root用户，运行以下命令

  ```
  # 删除用户
  userdel <username>
  
  # 删除用户，并删除用户的home目录
  userdel -r <username>
  ```

- 非root用户，需要添加`sudo`

### 参考

https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-a-centos-7-server

