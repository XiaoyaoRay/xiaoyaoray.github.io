---
title: AuthPoint添加ADFS
date: 2019-06-21 17:26:20
categories:
- WatchGuard
- ADFS
tags:
- ADFS
---

## WatchGuard Cloud添加ADFS Resource

### 进入到AuthPoint配置界面

在`Configure`中选择`AuthPoint`

![image-20190624102156098](http://ww2.sinaimg.cn/large/006tNc79gy1g4c16ygc00j30uu0gemyp.jpg)

### 添加ADFS Resource

- 在AuthPoint界面选择`Resource`， 选择`ADFS`并添加

![image-20190624102456597](http://ww2.sinaimg.cn/large/006tNc79gy1g4c1a15eqaj30s20f2jsp.jpg)

- 配置`ADFS`的名称，点击`SAVE`按钮

  ![image-20190624102609496](http://ww2.sinaimg.cn/large/006tNc79gy1g4c1bajv6ej31200b60t8.jpg)

- 在Gateway中添加`ADFS` Resource，并点击`SAVE`按钮保存

  ![image-20190624103440130](http://ww4.sinaimg.cn/large/006tNc79gy1g4c1k5j26oj30yr0u077g.jpg)

- 下载`ADFS Agent`

  ![image-20190624103836616](http://ww3.sinaimg.cn/large/006tNc79gy1g4c1o8z6v5j31ay0f6jta.jpg)



## Windows系统中安装并配置ADFS Agent

Windwos 系统中已经安装了`ADFS`服务，如果ADFS服务与Gateway在不同的系统中，**请关闭`Gateway`所在系统的防火墙**，否则ADFS Agent会安装失败。

==**以下所有操作均在Windows Server 2016 64bit中**==

### ADFS Agent安装

把ADFS Agent安装程序与配置文件放在**同一个目录**下，运行安装程序。然后点击`Next`一直等待安装完成

 ![image-20190624110318848](http://ww3.sinaimg.cn/large/006tNc79gy1g4c2dyhpm4j30du0at74n.jpg)

### 配置ADFS

- 打开ADFS 控制台

  ![image-20190624111810048](http://ww4.sinaimg.cn/large/006tNc79gy1g4c2texjd9j30sb0jkacv.jpg)

#### 添加WatchGuard认证

- 编辑`Multi-factor Authentication`， 对`Service`下`Authentication Methods`选项右键

  ![image-20190624112301990](http://ww3.sinaimg.cn/large/006tNc79gy1g4c2ygwbb9j30s30k10un.jpg)

- 选择`WatchGuard Multi Factor Authentication`，点击`OK`按钮

    ![image-20190624112501457](http://ww2.sinaimg.cn/large/006tNc79ly1g4c30klf9tj30di0hwmxi.jpg)

#### 添加Access Control Policy

- 选择`Access Control Policies`，右键添加`Policy`

  ![image-20190624114321871](http://ww4.sinaimg.cn/large/006tNc79gy1g4c3jn470vj30ry0jzwg4.jpg)
  
- 添加Rule

  ![image-20190624152209222](http://ww3.sinaimg.cn/large/006tNc79gy1g4c9va84wyj30s90jvq5c.jpg)

#### 应用已有Access Control Policy

- 在ADFS管理界面点击`Relying Party Trusts`，右键已有的`Relying Party Trusts`（如果没有请添加）选择`Edit Access Control Policy ...`

  ![image-20190624153326502](http://ww2.sinaimg.cn/large/006tNc79gy1g4ca70nfmcj30s90k1tan.jpg)

- 在编辑界面添加已有的Access Control Policy (New Access Control Policy)，点击`OK`按钮

  ![image-20190624154157961](http://ww2.sinaimg.cn/large/006tNc79gy1g4cafw0ph8j30rr0jaq54.jpg)
  
- 重启ADFS服务

  ![image-20190624161537667](http://ww2.sinaimg.cn/large/006tNc79gy1g4cbewxty3j30ry0ixabw.jpg)

### 访问ADFS页面

- `https://< domain >/adfs/ls/idpinitiatedsignon.aspx ` (ADFS服务所在的域名）或者`https://127.0.0.1/adfs/ls/idpinitiatedsignon.aspx`，**注意：选择MFA的Relying Party Trusts**

  ![image-20190624163658668](http://ww3.sinaimg.cn/large/006tNc79gy1g4cc14k71oj30s80kbdiw.jpg)

- 输入用户名和密码登录

  ![image-20190624163448547](http://ww2.sinaimg.cn/large/006tNc79gy1g4cbyvjkdvj30sb0kc77b.jpg)

- MFA认证

  ![image-20190624164040464](http://ww3.sinaimg.cn/large/006tNc79gy1g4cc5717a2j30s90katce.jpg)

## 参考

https://confluence.watchguard.com/display/WGQA/ADFS+agent