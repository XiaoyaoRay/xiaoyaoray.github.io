---
title: AuthPoint添加SAML
date: 2019-04-22 15:53:25
categories:
- WatchGuard
- SAML
tags:
- SAML
---

## FireBox Web UI配置

### WatchGuard Cloud UI上获取`METADAT URL`

打开WatchGuard Cloud UI(比如：[https://staging.cloud.watchguard.com](https://staging.cloud.watchguard.com/))，然后通过账户密码登录并进入到AuthPoint界面。

- 在`Resources`界面新建**Certificate**

![image-20190423144332795](https://image.rayleigo.com/markdown/202004/70bfbce4f4280d5bcf95a73261411e1b.jpg)

- 在`CERTIFICATE`界面上复制`METADATA URL`

  ![image-20190423144453467](https://tva1.sinaimg.cn/large/006tNc79gy1g2ckderf9fj31w80gwmz7.jpg)

### 登录FireBox UI

登录到FireBox Web UI(比如：https://< firebox ip >:8080)，然后通过账号和密码登录进入FireBox。

### FireBox Web UI配置Access Portal

在FireBox Web UI上选择**Subscription Services --> Access Portal**.可能需要先点击🔓

![image-20190422175756919](https://image.rayleigo.com/markdown/202004/3435d7eab366308cb11af1fa2d9d3e75.jpg)

#### 添加Applications

- 在**Applications中ADD按钮**添加SP(Service Provider)，比如：Web Application

![image-20190422180250765](https://image.rayleigo.com/markdown/202004/5bb25872385fb9070dca5516fe23673e.jpg)

- 配置Application的信息

![image-20190422180750195](https://image.rayleigo.com/markdown/202004/b038560fd9cdc40261860fd8796d778f.jpg)

#### 配置SAML

在Access Portal页面选择`SAML`页面

![image-20190422181014843](https://image.rayleigo.com/markdown/202004/cba23577b74ed78b45c57ff6b56891f8.jpg)

- 勾选`Enable Accesss Portal`
- 勾选`Enable SAML`
- 配置`Idp Name`，内容自定义。配置后再SAML登录界面会看到该信息
- 配置`Host Name`，内容自定义。注意：需要在本地的`hosts文件`把`FireBox`的IP地址与该Host Name对应(macOS中的`/etc/hosts`文件)
- 配置`Idp Metadata URL`，内容为从**WatchGuard Cloud UI**上复制到的`METADATA URL`数据(操作请见：**WatchGuard Cloud UI上获取`METADAT URL`**)
- 最后点击`SAVE`按钮

### FireBox Web UI配置Users and Groups

进入到`Users and Groups`界面，可能需要先点击下🔓

![image-20190422183117703](https://image.rayleigo.com/markdown/202004/6c04a3f1fccd39ba8973249d0cbb7e35.jpg)

#### 点击ADD按钮

![image-20190423094359746](https://image.rayleigo.com/markdown/202004/c9c329bf3bd2315235a481b6d6340c0e.jpg)

#### 配置用户信息

- type：选择`User`
- Name：自定义，注意需要在WatchGuard Cloud中有相应的用户才能正常使用
- Authentication Server：选择`AuthPoint`
- 其余的为默认值
- 点击`OK`保存

![image-20190423094542736](https://image.rayleigo.com/markdown/202004/c4ad6b7e87b3903cd543948848daa07a.jpg)

### 获取EntityID，Assertion Consumer Service(ACS) URL，Certificate

在浏览器中打开地址：https://< your Host Name >/auth/saml，其中`Host Name`为配置**SAML**时填的，并且该域名需要在`hosts文件`中配置

![image-20190423095313020](https://image.rayleigo.com/markdown/202004/3fe816337e0dbe77db5b46069278acb2.jpg)

## WatchGuard Cloud Web UI配置AuthPoint

### 进入到Resource界面

- 打开WatchGuard Cloud UI(比如：[https://staging.cloud.watchguard.com](https://staging.cloud.watchguard.com/))，并使用账户密码登录

![image-20190422172127535](https://image.rayleigo.com/markdown/202004/943b6e117fff84befee9a9fbe86ad7d9.jpg)

- 进入到AuthPoint界面

![image-20190422172345367](https://image.rayleigo.com/markdown/202004/5350d262716b3cc8fa6e1eb65ee4f11e.jpg)

- 进入到Resource界面

![image-20190423100444852](https://image.rayleigo.com/markdown/202004/7c074ffd360fedb3a4f5172a86bbc602.jpg)

### 添加SAML Resource

#### 选择SAML,并点击`Add Resource`按钮

![image-20190423100721545](https://image.rayleigo.com/markdown/202004/c6d1781673fdc5efc7442201ec7d1b6c.jpg)

#### 配置添加SAML信息

- **Name**：自定义
- **Application Type**: 选择Firebox Access Portal
- **Service Provider Entity ID** : 粘贴从`https://< your Host Name >/auth/saml`网页上复制的EntityID
- **Assertion Consumer Service**：粘贴从`https://< your Host Name >/auth/saml`网页上复制的Assertion Consumer Service URL
- **Choose File**：选择从`https://< your Host Name >/auth/saml`下载的证书
- 点击**SAVE**按钮

![image-20190423100846010](https://image.rayleigo.com/markdown/202004/0e87c9b03ae3c4db3e24a075de8d4f8c.jpg)

### 添加Group和User

#### 添加Group

- 点击Add Group按钮

![image-20190423103410439](https://image.rayleigo.com/markdown/202004/cb0682aefc9389c79ab6342850ade600.jpg)

- 添加Policy, 选择`SAML` Resource

![image-20190423103457822](https://image.rayleigo.com/markdown/202004/13a4710ecc37412287b4bd05a413a2b9.jpg)

- 点击**SAVE**按钮

#### 添加User

- 点击Add User按钮![image-20190423103605963](https://image.rayleigo.com/markdown/202004/1ce3109e6e35fa63b9950066a87b5a2e.jpg)

- 配置新用户信息，邮箱地址需要真实用户激活

  ![image-20190423103630457](https://image.rayleigo.com/markdown/202004/e717f9cc095b807a629e05128c7950ce.jpg)

#### 激活用户

使用添加用户的的邮箱，设置密码并用手机App扫邮件中二维码激活该该账户

### 登录用户

- SAML AuthPoint登录地址：http://< firebox ip >
- 登录界面输入用户
- 登录界面输入密码
- 手机App确认登录

