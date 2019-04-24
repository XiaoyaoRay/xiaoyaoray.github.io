---
title: 一步一步添加AuthPoint RADIUS
date: 2019-04-23 16:09:19
categories:
- WatchGuard
- RADIUS
tags:
- RADIUS
---

## Windows上安装Gateway

### 下载Gateway软件

下载完成后复制到Gateway的Windows主机上

地址： https://staging.deu.cloud.watchguard.com/services/auth/general/download

![image-20190424162937266](https://ws3.sinaimg.cn/large/006tNc79gy1g2dt0wdyx8j30yz09mt9n.jpg)

<!--more-->

### 获取Gateway Registration Key

[官网文档](https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/authpoint/gateway-registration-key.html)

- 点击添加Gateway

![image-20190424163443007](https://ws3.sinaimg.cn/large/006tNc79gy1g2dt5zgq6qj30fq07pjrp.jpg)

- 编辑Gateway界面获取`Gateway Registration Key`(**激活后不能再看到该Key**)

![image-20190424163602561](https://ws4.sinaimg.cn/large/006tNc79gy1g2dt7d4ibpj30xr0btgml.jpg)

### 系统中安装Gateway

安装时输入`Gateway Registration Key`

![img](https://ws4.sinaimg.cn/large/006tNc79gy1g2dtbyvz5pj30h408owfa.jpg)

## Firebox Web UI配置RADIUS

### 进入到RADIUS编辑界面

![image-20190424164257510](https://ws4.sinaimg.cn/large/006tNc79gy1g2dtek25d7j30n50ck3zl.jpg)

### 配置RADIUS信息

**需要打开编辑锁**

![image-20190424164434698](https://ws1.sinaimg.cn/large/006tNc79gy1g2dtg8gzt7j30rc0ezdgw.jpg)

- 打开`Enable RADIUS Server`
- IP address: Gateway所在主机的IP地址
- Port：1812(默认值)
- Share Secret： 自定义
- Confirm Secret： 确认`Share Secret`
- 其余看需求配置
- 点击`SAVE`保存配置

### 开启Firebox上4100端口访问

- 进入到Policies界面

![image-20190424164949941](https://ws3.sinaimg.cn/large/006tNc79gy1g2dtlpnagqj312509jgny.jpg)

- 开启访问

![image-20190424165029086](https://ws1.sinaimg.cn/large/006tNc79gy1g2dtmdwfobj315m0f0myd.jpg)

## WatchGuard Cloud Web UI配置

### 添加RADIUS Client

#### 点击`Add Resource`按钮

![image-20190424165305479](https://ws4.sinaimg.cn/large/006tNc79gy1g2dtp3riwkj30k10893z1.jpg)

#### 配置RADIUS Client信息

![image-20190424165351293](https://ws2.sinaimg.cn/large/006tNc79gy1g2dtpw995cj30q809v3yv.jpg)

- Name： 自定义
- RADIUS client trusted IP or FQDN：Firebox的IP地址
- Shared Secret ：Firebox配置RADIUS时配置的**Secret**
- 点击`SAVE`按钮保存

### RADIUS Client绑定到Gateway

![image-20190424165657497](https://ws3.sinaimg.cn/large/006tNc79gy1g2dtt4gf61j30q40bh0t5.jpg)

- Port： Firebox配置RADIU时配置的端口
- Select a RADIUS resource：选择添加的`RADIUS Client`

### 添加Group和User

#### 添加Group

- 点击Add Group按钮

![image-20190423103410439](https://ws2.sinaimg.cn/large/006tNc79gy1g2cd4izkdqj30h30c3js7.jpg)

- 添加Policy

![image-20190423103457822](https://ws1.sinaimg.cn/large/006tNc79gy1g2cd5cg9h5j30qg0ld0ts.jpg)

- 点击**SAVE**按钮

#### 添加User

- 点击Add User按钮

![image-20190423103605963](https://ws4.sinaimg.cn/large/006tNc79gy1g2cd6j3n4gj30iq0b6js6.jpg)

- 配置新用户信息，邮箱地址需要真实用户激活

![image-20190423103630457](https://ws4.sinaimg.cn/large/006tNc79gy1g2cd6yesy4j30q40hiwf5.jpg)

#### 激活用户

使用添加用户的的邮箱，设置密码并用手机App扫邮件中二维码激活该该账户

### 登录用户

- SAML AuthPoint登录地址：http://< firebox ip >:4100
- 登录界面输入用户
- 登录界面输入密码
- 手机App确认登录