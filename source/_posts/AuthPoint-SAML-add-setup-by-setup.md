---
title: 一步一步添加AuthPoint SAML
date: 2019-04-22 15:53:25
categories:
- WatchGuard
- SAML
tags:
- SAML
---

## FireBox Web UI配置

### WatchGuard Cloud UI上获取`METADAT URL`

- 打开WatchGuard Cloud UI(比如：[https://staging.cloud.watchguard.com](https://staging.cloud.watchguard.com/))，并使用账户密码登录

![image-20190422172127535](https://ws4.sinaimg.cn/large/006tNc79gy1g2bja38nwej30pe0dn3yo.jpg)

<!--more-->

- 进入到AuthPoint界面

![image-20190422172345367](https://ws2.sinaimg.cn/large/006tNc79gy1g2bjcgvsm7j30cq07ajrj.jpg)

- 在`Resources`界面新建**Certificate**

![image-20190423144332795](https://ws3.sinaimg.cn/large/006tNc79gy1g2ckc0bv2nj30va0dg3zj.jpg)

- 在`CERTIFICATE`界面上复制`METADATA URL`

  ![image-20190423144453467](https://ws3.sinaimg.cn/large/006tNc79gy1g2ckderf9fj31w80gwmz7.jpg)

### 登录FireBox UI

- 登录到FireBox Web UI(比如：https://< firebox ip >:8080)

![image-20190422175256407](https://ws2.sinaimg.cn/large/006tNc79gy1g2bk6tvmutj309707i749.jpg)

### FireBox Web UI配置Access Portal

在FireBox Web UI上选择**Subscription Services --> Access Portal**.可能需要先点击🔓

![image-20190422175756919](https://ws1.sinaimg.cn/large/006tNc79gy1g2bkc0tmlbj30gy08hq3n.jpg)

#### 添加Applications

- 在**Applications中ADD按钮**添加SP(Service Provider)，比如：Web Application

![image-20190422180250765](https://ws1.sinaimg.cn/large/006tNc79gy1g2bkh4ey1ej30ft0frmxw.jpg)

- 配置Application的信息

![image-20190422180750195](https://ws1.sinaimg.cn/large/006tNc79gy1g2bkmb6jwpj30gb0hbq3o.jpg)

#### 配置SAML

在Access Portal页面选择`SAML`页面

![image-20190422181014843](https://ws4.sinaimg.cn/large/006tNc79gy1g2bkotwqcqj30pf0k5mz4.jpg)

- 勾选`Enable Accesss Portal`
- 勾选`Enable SAML`
- 配置`Idp Name`，内容自定义。配置后再SAML登录界面会看到该信息
- 配置`Host Name`，内容自定义。注意：需要在本地的`hosts文件`把`FireBox`的IP地址与该Host Name对应(macOS中的`/etc/hosts`文件)
- 配置`Idp Metadata URL`，内容为从**WatchGuard Cloud UI**上复制到的`METADATA URL`数据(操作请见：**WatchGuard Cloud UI上获取`METADAT URL`**)
- 最后点击`SAVE`按钮

### FireBox Web UI配置Users and Groups

进入到`Users and Groups`界面，可能需要先点击下🔓

![image-20190422183117703](https://ws2.sinaimg.cn/large/006tNc79gy1g2blaq4d7rj30il0b23z7.jpg)

#### 点击ADD按钮

![image-20190423094359746](https://ws2.sinaimg.cn/large/006tNc79gy1g2cbobdcqzj30ap04m3yi.jpg)

#### 配置用户信息

- type：选择`User`
- Name：自定义，注意需要在WatchGuard Cloud中有相应的用户才能正常使用
- Authentication Server：选择`AuthPoint`
- 其余的为默认值
- 点击`OK`保存

![image-20190423094542736](https://ws4.sinaimg.cn/large/006tNc79gy1g2cbq3qggjj30fj0ce3zc.jpg)

### 获取EntityID，Assertion Consumer Service(ACS) URL，Certificate

在浏览器中打开地址：https://< your Host Name >/auth/saml，其中`Host Name`为配置**SAML**时填的，并且该域名需要在`hosts文件`中配置

![image-20190423095313020](https://ws1.sinaimg.cn/large/006tNc79gy1g2cbxx61ooj30en0o7juq.jpg)

## WatchGuard Cloud Web UI配置AuthPoint

### 进入到Resource界面

- 打开WatchGuard Cloud UI(比如：[https://staging.cloud.watchguard.com](https://staging.cloud.watchguard.com/))，并使用账户密码登录

![image-20190422172127535](https://ws4.sinaimg.cn/large/006tNc79gy1g2bja38nwej30pe0dn3yo.jpg)

- 进入到AuthPoint界面

![image-20190422172345367](https://ws2.sinaimg.cn/large/006tNc79gy1g2bjcgvsm7j30cq07ajrj.jpg)

- 进入到Resource界面

![image-20190423100444852](https://ws2.sinaimg.cn/large/006tNc79gy1g2cc9wqu6kj30j909474s.jpg)

### 添加SAML Resource

#### 选择SAML,并点击`Add Resource`按钮

![image-20190423100721545](https://ws1.sinaimg.cn/large/006tNc79gy1g2cccmiblcj30fr095dg2.jpg)

#### 配置添加SAML信息

- **Name**：自定义
- **Application Type**: 选择Firebox Access Portal
- **Service Provider Entity ID** : 粘贴从https://< your Host Name >/auth/saml网页上复制的EntityID
- **Assertion Consumer Service**：粘贴从https://< your Host Name >/auth/saml网页上复制的Assertion Consumer Service URL
- **Choose File**：选择从**Choose File**下载的证书
- 点击**SAVE**按钮

![image-20190423100846010](https://ws3.sinaimg.cn/large/006tNc79gy1g2cce3ogjjj30px0nqta1.jpg)

### 添加Group和User

#### 添加Group

- 点击Add Group按钮

![image-20190423103410439](https://ws2.sinaimg.cn/large/006tNc79gy1g2cd4izkdqj30h30c3js7.jpg)

- 添加Policy, 选择`SAML` Resource

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

- SAML AuthPoint登录地址：http://< firebox ip >
- 登录界面输入用户
- 登录界面输入密码
- 手机App确认登录

