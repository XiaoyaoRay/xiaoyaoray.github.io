---
title: AuthPoint添加LogonApp
date: 2019-04-28 17:07:02
categories:
- WatchGuard
- LogonApp
tags:
- LogonApp
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



## WatchGuard Cloud Web UI 配置

### 添加Logon App Resource

#### 点击`Add Resource`按钮

![image-20190428171256818](https://ws2.sinaimg.cn/large/006tNc79gy1g2igr1k6bxj30gn06gt94.jpg)

<!--more-->

#### 配置Logon App信息

![image-20190428171343817](https://ws3.sinaimg.cn/large/006tNc79gy1g2igru3upej30ny0cy0t8.jpg)

- Name: 自定义
- Support Message: 自定义，最后可以在Windows的登录界面上看到
- Synchronization Interval (Hour)： 自定义
- Remember Password：自定义，可以勾选或者不勾选

## Logon App 安装

### 下载Logon App

需要下载`INSTALLER`和`CONFIG`

![image-20190428172211122](https://ws1.sinaimg.cn/large/006tNc79gy1g2ih0lsd0zj30xo0fftap.jpg)

### 系统中安装Logon App

把安装程序和配置文件放在同一目录下，运行安装程序。安装完成后由下图提示，重启系统后需要通过AuthPoint来登录(**注意**：系统的本地账户在WatchGuard Cloud有**相同的user**才能登录)

![image-20190428172856853](https://ws3.sinaimg.cn/large/006tNc79gy1g2ih7n1ofej30is08mdii.jpg)

## 添加Group和User

### 添加Group

- 点击Add Group按钮

![image-20190423103410439](https://ws2.sinaimg.cn/large/006tNc79gy1g2cd4izkdqj30h30c3js7.jpg)

- 添加Policy, 选择`Logon App`

![image-20190423103457822](https://ws1.sinaimg.cn/large/006tNc79gy1g2cd5cg9h5j30qg0ld0ts.jpg)

- 点击**SAVE**按钮

### 本地账户

#### Windows系统创建本地账户

添加步骤请见链接：https://ibooks.red/2019/04/28/Windows10-add-local-user

#### WatchGuard Cloud添加相同账户

**注意：**添加时用户名要与Windows系统中添加的名称**一样**

![image-20190428175347825](https://ws2.sinaimg.cn/large/006tNc79gy1g2ihxhxw0vj30hp0bmjsa.jpg)

#### 激活用户

使用添加用户的的邮箱，设置密码并用手机App扫邮件中二维码激活该该账户

#### 登录用户

在Windows登录界面操作

- 登录界面输入用户
- 登录界面输入密码
- 手机App确认登录

### LDAP用户登录

#### 添加LDAP服务

- 点击`Add External Identity`按钮

![image-20190429093403093](https://ws3.sinaimg.cn/large/006tNc79gy1g2j93v3njej30ih07o3yx.jpg)

- 配置LDAP服务信息
  - Name: 自定义
  - LDAP Search Base: 根据LDAP服务器信息配置(比如： dc=jamesad,dc=com)
  - System Account: LDAP服务器账号(比如： administrator)
  - Passphrase： system account的的密码
  - Synchronization interval:  根据需求选择
  - Domain: LDAP服务器域名
  - Attribute related to the first name: 根据需求配置，默认值`givenName`
  - Attribute related to the last name: 根据需求配置，默认值`sn`
  - Attribute related to the user email: 根据需求配置，默认值`mail`
  - Main attribute to the LDAP user: 根据需求配置，默认值`objectGUID`
  - Attribute related to the user login: 根据需求配置，默认值`sAMAccountName`
  - Attribute related to the mobile number: 根据需求配置，默认值`mobile`
  - Server Address: LDAP服务器IP地址
  - Server Port: 根据需求配置，默认值`389`
  - 点击`SAVE`按钮

![image-20190429093525927](https://ws3.sinaimg.cn/large/006tNc79gy1g2j959nl1aj30pc0mzmyo.jpg)

- LDAP服务绑定到Gateway中

![image-20190429095020825](https://ws3.sinaimg.cn/large/006tNc79gy1g2j9ksa6m9j30p60nqgn1.jpg)



#### 添加Query同步LDAP到LogonApp用户组中

![image-20190428175830934](https://ws4.sinaimg.cn/large/006tNc79gy1g2ii2edw6aj314n08e3zg.jpg)

#### Windows系统中配置Domain

- 添加DNS：把`LDAP`服务地址输入到DNS server中

![image-20190428143554139](https://ws3.sinaimg.cn/large/006tNc79gy1g2ii57f032j30ku0nm7ak.jpg)

- 在系统属性中配置，在`Domain`中输入LDAP服务中的域名，**注意：**添加时会验证LDAP的用户，即输入LDAP服务器的用户名和密码

![image-20190428143451835](https://ws2.sinaimg.cn/large/006tNc79gy1g2ii3je93dj31360okgwa.jpg)

#### 激活用户

使用添加用户的的邮箱，设置密码并用手机App扫邮件中二维码激活该该账户

#### 登录用户

在Windows登录界面操作

- 登录界面输入用户
- 登录界面输入密码
- 手机App确认登录