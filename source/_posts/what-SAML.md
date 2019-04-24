---
title: SAML基础学习
date: 2019-04-22 13:50:13
categories:
- WatchGuard
- SAML
tags:
- SAML
---

## 转载：[SAML Web SSO学习](<https://blog.csdn.net/xiangguiwang/article/details/66476171>)

## SAML名称解释

SAML即安全声明标记语言，英文全称是Security Assertion Markup Language. 它是一个基于XML的标准，用于在不同的安全域(SecurityDomain)之间认证和授权数据。在SAML标准定义了身份提供者(IdentityProvider)和服务提供商(ServiceProvider)，这两者构成了前面所说的不同的安全域。SAML是OASIS组织安全服务技术委员会(Security Services Technical Committee)的产品。

<!--more-->

## SAML组成

SAML2.0由断言(Assertions)、协议(Protocols)、绑定(Bindings)、配置(Profiles)、Metadata组成。

![image-20190422162402835](https://ws2.sinaimg.cn/large/006tNc79gy1g2bhmchxxnj30nb0ea0ub.jpg)



### 断言(Assertions)：认证，属性和授权信息。

SAML定义了可以在断言中携带的三种语句：

- 认证语句（Authenticationstatements）：这些由成功认证用户的一方创建。至少，它们描述了用于认证用户的特定方式以及认证发生的具体时间。
  
- 属性语句（Attributestatements)们包含关于主题的特定标识属性（例如，用户“John Doe”具有“Gold”卡状态）。
  
- 授权决策语句（Authorizationdecision statements）：这些定义了主题有权做的事情.（例如，“John Doe”是否被允许购买指定项目）。

### 协议(Protocols)：获取断言和进行身份管理的请求和响应。
（Authentication Request Protocol）认证请求协议：定义主体（或代表主体的代理）
以请求包含认证语句和可选的属性语句的断言的方法。
例如：SingleLogout Protocol、Assertion Query and Request Protocol、ArtifactResolution Protocol、Name Identifier Management Protocol、NameIdentifier Mapping Protocol。

### 绑定(Bindings)：SAML协议的映射到标准消息和通信协议。
例如：HTTPRedirect Binding、HTTP POST Binding、HTTP Artifact Binding、SAML SOAPBinding、Reverse SOAP (PAOS) Binding、SAML URIBinding。
### 配置(Profiles)：由断言，协议，和绑定组成的用例。
例如：WebBrowser SSO Profile、Single Logout Profile。
Metadata:idp和sp的配置数据。Metadata文件定义了在SAML方之间表达和共享配置信息的方式。例如，可以使用Metadata XML文档来表达实体支持的SAML绑定，操作角色（IDP，SP等），标识符信息，支持身份属性以及用于加密和签名的密钥信息。
### MetaData
我们来看看Metadata的一些重要用途：

- IDP通过浏览器从SP接收<samlp：AuthnRequest>元素。IDP如何知道SP是真实的，而不是一些恶意的SP窃取有关用户的私人信息？在发出身份验证响应之前，IDP会在元数据中查询受信SP的列表。
- 在上述的情况下，IDP如何知道将用身份验证响应返回到哪？IDP在Metadata中查找SP的预先安排的端点位置。
- SP如何知道验证响应来自受信任的IDP？SP使用来自MetaData的IDP的公钥来验证断言上的签名。 
- SP如何知道从受信任的IDP解析组件的位置？SP从MetaData查找IDP的组件服务的预先安排的端点位置。
- 总而言之，MetaData确保IDP和SP之间的安全交易。在MetaData之前，信任信息以专有方式编码到实现中。 现在，通过标准Metadata来促进信任信息的共享。SAML 2.0提供了一个定义良好的可互操作的MetaData格式，实体可以利用该格式来引导信任过程。
#### Metadata 重要节点
Element < EntitiesDescriptor >
SAML元数据实例描述单个实体或多个实体。
在前一种情况下，根元素必须是< EntityDescriptor >。 在后一种情况下，根元素必须是< EntitiesDescriptor >
< EntitiesDescriptor >元素包含可选命名的SAML实体组的元数据。 它的EntitiesDescriptorType复杂类型包含一个< EntityDescriptor >元素，< EntitiesDescriptor >元素或两者的序列：

- ID [可选] 元素唯一标识符，通常在签名时用作参考点。
- validUntil [可选] 可选属性指示元素中包含的元数据和任何包含的元素的到期时间。
-  cacheDuration [可选] 可选属性表示用户应缓存元素中包含的元数据和任何包含的元素的最大时间长度。
- entityID  实体ID
- <ds：Signature> [可选] 用于验证包含元素及其内容的XML签名
- XML签名[XMLSig]定义了<ds：KeyInfo>元素的用法。 SAML不要求使用<ds：KeyInfo>，也不对其使用施加任何限制。 因此，<ds：KeyInfo>可能不存在。
## SAML SSO流程
### SP-Initiated SSO: Redirect/POST Bindings
![image-20190423090452128](https://ws4.sinaimg.cn/large/006tNc79gy1g2cb862xpvj30mq0iigq8.jpg)



        第一步：

              用户尝试访问sp.example.com上的资源。Sp检查用户在此网站上是否有登录会话（即安全上下文）。如果用户在该网站上有登录会话，则SP直接让用户访问资源。否则，SP将发起SSO流程。

       第二步：

              SP发送一个HTTP消息返回给浏览器，HTTP消息中包含AuthnRequest消息。HTTP消息是URL encoded。

       第三步：

              提示用户输入认证。认证方式有用户名+口令。

       第四步：

              用户提交认证信息登录。

       第五步：

               IDP提供用户的权限信息。

       第六步：

              SP解析用户的权限信息并决定返回用户访问的资源页面。

       第七步：

              用户访问应用。

### SP-Initiated SSO: POST/Artifact Bindings

        第一步：

              用户尝试访问sp.example.com上的资源。Sp检查用户在此网站上是否有登录会话（即安全上下文）。如果用户在该网站上有登录会话，则SP直接让用户访问资源。否则，SP将发起SSO流程。

       第二步：

              SP发送一个HTTP消息返回给浏览器，HTTP消息中包含AuthnRequest消息。HTTP消息是URL encoded。

       第三步：

              提示用户输入认证。认证方式有用户名+口令。

       第四步：

              用户提交认证信息登录。

       第五步：

              IDP提供用户的权限信息。

       第六步：

              SP根据artifact ID获取权限。

       第七步：

              ID返回权限信息。

       第八步：

              用户访问应用。

### IdP-Initiated SSO: POST Binding

        第一步：

              IDP提示用户输入认证。认证方式有用户名+口令。

       第二步：

              用户提交认证信息登录。

       第三步：

              选择用户访问的应用。

       第四步：

              把用户授权信息发送给SP。

       第五步：

              浏览器重定向授权信息给SP。

       第六步：

              用户访问应用。

### AuthenRequest
```
<samlp:AuthnRequest
xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" ID="ONELOGIN_809707f0030a5d00620c9d9df97f627afe9dcc24" 
Version="2.0" 
ProviderName="SP test" 
IssueInstant="2014-07-16T23:52:45Z"
 Destination="http://idp.example.com/SSOService.php" ProtocolBinding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" AssertionConsumerServiceURL="http://sp.example.com/demo1/index.php?acs">
<saml:Issuer>http://sp.example.com/demo1/metadata.php</saml:Issuer>
  <samlp:NameIDPolicy
 Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress" AllowCreate="true"/>
  <samlp:RequestedAuthnContext Comparison="exact">
<saml:AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport</saml:AuthnContextClassRef>	
  </samlp:RequestedAuthnContext>
</samlp:AuthnRequest>
```
### AuthnResponse

```
<samlp:Response
 xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" ID="_8e8dc5f69a98cc4c1ff3427e5ce34606fd672f91e6"
 Version="2.0" 
IssueInstant="2014-07-17T01:01:48Z" Destination="http://sp.example.com/demo1/index.php?acs" InResponseTo="ONELOGIN_4fee3b046395c4e751011e97f8900b5273d56685">
  <saml:Issuer>http://idp.example.com/metadata.php</saml:Issuer>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>
  </samlp:Status>
  <saml:Assertion 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xs="http://www.w3.org/2001/XMLSchema" ID="_d71a3a8e9fcc45c9e9d248ef7049393fc8f04e5f75" Version="2.0" IssueInstant="2014-07-17T01:01:48Z">
    <saml:Issuer>http://idp.example.com/metadata.php</saml:Issuer>
    <saml:Subject>
      <saml:NameID 
SPNameQualifier="http://sp.example.com/demo1/metadata.php" 
Format="urn:oasis:names:tc:SAML:2.0:nameid-format:transient">_ce3d2948b4cf20146dee0a0b3dd6f69b6cf86f62d7</saml:NameID>
      <saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <saml:SubjectConfirmationData
 NotOnOrAfter="2024-01-18T06:21:48Z" Recipient="http://sp.example.com/demo1/index.php?acs" InResponseTo="ONELOGIN_4fee3b046395c4e751011e97f8900b5273d56685"/>
      </saml:SubjectConfirmation>
    </saml:Subject>
<saml:Conditions 
NotBefore="2014-07-17T01:01:18Z" NotOnOrAfter="2024-01-18T06:21:48Z">
      <saml:AudienceRestriction>
        <saml:Audience>http://sp.example.com/demo1/metadata.php</saml:Audience>
      </saml:AudienceRestriction>
    </saml:Conditions>
<saml:AuthnStatement
 AuthnInstant="2014-07-17T01:01:48Z" 
SessionNotOnOrAfter="2024-07-17T09:01:48Z" SessionIndex="_be9967abd904ddcae3c0eb4189adbe3f71e327cf93">
      <saml:AuthnContext>
        <saml:AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:Password</saml:AuthnContextClassRef>
      </saml:AuthnContext>
    </saml:AuthnStatement>
    <saml:AttributeStatement>
      <saml:Attribute 
Name="uid" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:basic">
        <saml:AttributeValue xsi:type="xs:string">test</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute 
Name="mail" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:basic">
        <saml:AttributeValue xsi:type="xs:string">test@example.com</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute 
Name="eduPersonAffiliation" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:basic">
        <saml:AttributeValue xsi:type="xs:string">users</saml:AttributeValue>
        <saml:AttributeValue xsi:type="xs:string">examplerole1</saml:AttributeValue>
      </saml:Attribute>
    </saml:AttributeStatement>
  </saml:Assertion>
</samlp:Response>
```

### SP 处理AuthnResponse步骤

无论使用何种SAML绑定，SP都必须执行以下操作：

- 验证断言或响应上存在的任何签名
- 验证属性< SubjectConfirmationData >是否与提供< Response >Destination URL相匹配
- 验证在任何承载< SubjectConfirmationData >中的NotOnOrAfter属性是否过期。 
- 验证承载中的InResponseTo属性< SubjectConfirmationData >等于其原始< AuthnRequest >消息的ID，除非响应是未经请求的，在这种情况下，属性不能存在
- 如果用于建立主体的安全上下文的< AuthnStatement >包含一个SessionNotOnOrAfter属性，
- 获取< AttributeStatement >中的权限信息。

## SAML Single Logout流程
本用例讨论一个代表性流程选项：在一个SP处发起的单个注销，并且导致从多个SP注销。
![image-20190423092804342](https://ws4.sinaimg.cn/large/006tNc79gy1g2cb7r1chmj30kn0fpjuf.jpg)

### LogoutRequest

```
<samlp:LogoutRequest
 xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
 xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" ID="ONELOGIN_21df91a89767879fc0f7df6a1490c6000c81644d"
 Version="2.0" 
IssueInstant="2014-07-18T01:13:06Z" Destination="http://idp.example.com/SingleLogoutService.php">
 <saml:Issuer>http://sp.example.com/demo1/metadata.php</saml:Issuer>
 <saml:NameID
 SPNameQualifier="http://sp.example.com/demo1/metadata.php" Format="urn:oasis:names:tc:SAML:2.0:nameid-format:transient">ONELOGIN_f92cc1834efc0f73e9c09f482fce80037a6251e7
</saml:NameID>
</samlp:LogoutRequest>
```

### LogoutResponse

```<samlp:LogoutResponse 
xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" 
xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" 
ID="_6c3737282f007720e736f0f4028feed8cb9b40291c" 
Version="2.0" 
IssueInstant="2014-07-18T01:13:06Z" 
Destination="http://sp.example.com/demo1/index.php?acs" InResponseTo="ONELOGIN_21df91a89767879fc0f7df6a1490c6000c81644d">
  <saml:Issuer>http://idp.example.com/metadata.php</saml:Issuer>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>
  </samlp:Status>
</samlp:LogoutResponse>
```

## 参考资料
- https://en.wikipedia.org/wiki/SAML_2.0 
- http://download.csdn.net/detail/xiangguiwang/9791789
- http://download.csdn.net/detail/xiangguiwang/9791808