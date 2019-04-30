---
title: WatchGuard-firebox-tranining
date: 2019-04-30 18:11:18
categories:
- WatchGuard
- Firebox
tags:
- Firebox
---

# FireBox Training

## 设备

SN,PN,KVM均为自动化准备的

### SN

Serial over IP deice

- 17个端口：一个提供给用户访问SN设备，其余16个端口为网络接口(管理不同设备的串口)

<!--more-->

### PN

Power over IP deice

- 类似SN管理工具，主要是控制设备的电源

### KVM

多台设备连接一个显示器即显示器可以切换到不同的设备

### 串口

- 使用需要在本机安装驱动（USB转串口） 
- 设备波特率：115200 
- 稳定可靠

### Hardware Platform：

[Confluence上地址](https://confluence.watchguard.com/pages/viewpage.action?spaceKey=WGQA&title=Hardware+Platform)

XTM系列已经不再维护，被T,M,FirefoxV替换，FirefoxV软件(类似虚拟机)。设备型号中-W 带无线，-D DSL猫拨号，-F带光纤（没有F也有可能带光纤）

- RTM： 出厂时第一次版本号（同型号设备可能不同）
- X86 vs PPC： CPU平台，比如：PPC 64，x86 64
- POE：网络连接或者供电(即某个插口同时又网络功能和供电功能)
- Crypto： 加密芯片，比如：internal，CCK8903
- T开头的型号：小盒子
- M开头的型号：大盒子



### 测试软件：

#### 管理工具

- WebUI

  https://172.23.0.233:8080

- CLI

- WSM

  只能去西雅图的image server下载: http://cmimages.wgti.net/

  用于管理Firebox的client端

  其中PM: 能管理配置文件和License，修改需要保存才能生效，也可以修改多处后再保存

- UTM

#### 账号

- status: readonly

  只有读取权限

- admin: readwrite

  可读可写

- root

  debug版本能进入root账户（非默认开启），在admin用户下执行：diagnose enable readwrite(密码，可以自定义)

#### 版本切换

- 测试使用release版本，不推荐使用debug版本，sysa.dl升级经常使用
- Firebox中有两套操作系统，sysb系统不需要最新的（需要保持稳定）
- release版本到debug版本：在链接申请访问权限，或者使用package升级。

### 敏捷与瀑布开发

#### 瀑布

- 按计划一整套来做（设计，开发，测试）<-- Firebox开发，所有功能都有联系不能很好拆分
- mainline的开发主线（开发工具：perforce即p4，每次有chagelist即CL）
- 其他自动化开发工具：git

#### 敏捷

- 切成小任务，更快速，更灵活

#### Perforce(即p4)

- 拉取Branch，image server的文件夹对应perforce的不同分支
- task branch: 开发单独的功能，image server的文件夹名最后带字母的

#### Bug

- needmeage，需要在mainline和v1 release两个分支上验证
- JIRA 上查看fix version
- changelist的位置是否大于fix branch： 大于且不再fix branch上需要验证多个分支的

### License

  每一个平台，每一种型号都有对应的定义，一个设备一个License，[LICENSE SPECS](https://wgt.sharepoint.com/engineering/Engineering%20Document%20Library/Forms/AllItems.aspx?RootFolder=%2Fengineering%2FEngineering%20Document%20Library%2F_LICENSE%20SPECS&FolderCTID=0x01200016762C401C4A2642B6F0EDAD8DEC76CA&View=%7BB9CC7568-B695-4BE2-AD86-AA2D4E4D4A8F%7D)

#### License 存放位置

- /etc/wg/license: Activation License
- /etc/license: Default License

#### License 说明

- Feature: 时间或者数量，即按时间和数量收费。数量是指接入设备允许使用的个数（比如：VPN的用户使用个数）
- Signature: 验证有效性
- Pro: 套餐，与其他License取最大
- DevKey: 使用默认值，除非很清楚修改内容（比如性能测试）

#### LIVESECURITY: 升级 

- 升级需要转换配置文件
- build中release-date与LIVESECURITY对比，过期
- releaseDate: 发布时间，测试的时候还未发布，是我们定义的时间
- 客户的License：根据SN 和 SKU生成，在官网查看

#### SKU

- 最小可售单元
- `FB`或者`FB+L`或者` L` (FB: Firebox，L: License)

#### MSSP

- 用户可自己配置，根据账号余额来使用
- Standard Support： 基本网络功能

#### 降级

- 以保存了以前配置，出厂恢复
- 官方不支持降级

#### 恢复FireBox

- Restore factory-default 清除配置文件
- Restore factory-default all 清除所有的用户数据：管理员密码，配置文件（设备上reset按键等同）
- Forgot password： 进入到 Boot info safe mode --> QSW 设备直连（相同网段），使用root账号或admin账号恢复密码

### 学习

- TCP/IP协议卷一/二
- IPTables NetFilter架构
- 鸟哥的私房菜

### 连接FireBox工具

- PuTTY
- mPuTTY
- SN管理设备: Telnet方式连接，自定义端口（在SN上配置）
- SSH: 4118端口
- WEB: 8080端口

### FireBox中文件

- /etc/wg: 配置文件，Firebox以及其他配置文件
- config.xml: 压缩文件(PE头决定文件真正格式), 备份 gzip解压 修改   
- certs: 存放证书
- backup: 对系统备份，恢复系统到某个时间点
- upgrade.txt: 对版本升级操作记录
- /var/log: 系统log
- /tmp/debug: 进程的log
- 查看log： logread -f 或者 tail -f

### wgcmd 命令

- 发给wgagent进程，使用xpath来区别
- wgcmd status /system/info 查看系统信息 --> 相当于进程wgagent --> 相当于进程system
- wgcmd status /authentication/list 获取当前登录用户
- wgcmd action /authentication/logoff type=99 剔除当前登录用户，不剔除root账户（剔除status，admin账号）
- wgcmd status /network --> 获取帮助
- 中间状态使用wgcmd来模拟

### Network

#### 交换机

- 二层
- 有mac 
  - 根据mac来转发 --> mac table
  - src mac:  mac table中没有将会记录下来 
  - dst mac:  查看mac table

#### 路由器

- 三层
- 有IP 
  - 根据IP来转发 --> routing table
  - 路由查询最为关键，每条均会路由查询
  - 0.0.0.0/0: 默认路由
  - 每一个端口对应一个IP地址
  - 子网掩码： 区分网络段和主机段，根据IP地址来区分
  - src ip: routing table中没有将会记录下来 
  - dst ip:  查看routing table --> 先最精确匹配（范围最小），再metrc(开销)最小，精确匹配优先级更高--> C直连路由

### 网络模式

- Mixed Routing Mode == 3 层 90%客户
- Bridage Mode == transparent mode 2层 --> 已有Router，添加防火墙且不影响原有网络（添加安全策略）
- Drop In Mode == 2或者3层

#### NAT

 网络地址转换

- DNAT: 公司 dgnamic NAT --> src NAT --> 转化源IP地址 --> 多对一
- SNAT: 公司 static NAT --> dst NAT --> 转化目的IP地址 --> 一对一
- 1-to-1 NAT: src NAT + dst NAT --> 内部的IP地址与外部IP地址绑定 --> 双向的
- Server load balance(SLB): dst NAT --> 多台目的服务 --> 多个目的IP地址 --> 一对多

#### 发送：

- 目的IP不变
- 在路由表中查看下一跳的目的MAC，封装src mac，dst mac
- 第一次发送时创建Comutrack或者Session表第一次发送时创建Comutrack或者Session表

#### 回包

- 回来的使用Session表,不查policy

### 防火墙

- Session查询最为关键，先查Session表，在查policy：即第一条数据包查policy，其余的均Session表。查policy及其费性能
- 防火墙默认不允许任何网络通关，添加策略通过某些网络
- 策略：配置才生效，如果没有勾选没有意义
  - 来源： 目的端口，源用户，源ip，源域名，源网络段等等
  - 顺序匹配: 匹配到第一条，后面的不关心 -- 顺序自定义或系统自动匹配（越精确越往上放即顺序越靠前）
- 所做的两类事件
  - 检测数据包（单个，一个链接，某个IP地址，端口，协议等）
  - 检测到后做某些动作（进一步检测等）
- 认证（Authenlication）:  把IP地址与人结合起来，然后对不同的人对不同的操作

### VPN

- 专用虚拟网络
- local与remote要对等（证书，密码一样）
- IKE/IPSEC: `BoVPN`和`BoVPNVIF `，端口：500/4500
- TLS-SSL:  `BoVPNVIF` ，端口 ：443 ，必须由client端发起建立连接

### Cluster

- master
- backupmaster
- ActivePassive：使用虚MAC，把master与backupmaster的MAC改为一样，让一个IP对应一个MAC地址

### Proxy&Packet Filtters：

流扫描，会缓存进行扫描，配置扫描范围，绝大部分病毒都在数据包前面字段

### EICAR 病毒引擎检测字段 