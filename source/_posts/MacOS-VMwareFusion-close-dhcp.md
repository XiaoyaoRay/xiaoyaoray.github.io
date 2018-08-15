---
title: Mac VMware fusion 网络关闭DHCP
copyright: true
date: 2018-08-08 10:54:25
categories:
- MacOS
tags:
- MacOS
---

### 转载：[Mac VMware fusion 专用网络关闭DHCP](https://blog.csdn.net/miiser/article/details/44419759)

因为工作需要所以需要安装虚拟机来测试，学习使用Panabit流控系统的常用功能，所以添加了很多的虚拟机的网卡，但是每张网卡的使用方式又不一样，需要一个控制口，一个外网口，一个内网口。

<!--more-->

- 控制口使用静态地址，这里用的Mac新建的网络配置，然后虚拟机桥接到该网络配置，使其在同一网段。
- 外网口桥接在以太网配置，使其作为一台物理机来使用上级路由器的ip，可以正常访问外网。
- 内网口则使用的专用网络方式，而且在panabit上面开启了DHCP服务，所以虚拟机的专用网络vmnet自带的DHCP功能，导致我新建的xp虚拟机来作为客户机没法获取到panabit分配的ip。

所以我在网上也没有找到相关的信息，之前使用Windows的时候Vmware有一个虚拟网络编辑器，可以直接关闭DHCP比较方便。

但是我这里又想关闭DHCP，所以找到的方法如下

进入相关目录

```
 cd /Library/Preferences/VMware\ Fusion/1
```

编辑文件

```
sudo vim networking1
```

修改文件

```
VERSION=1,0
answer VNET_1_DHCP no
answer VNET_1_DHCP_CFG_HASH 5D88992D00D27DF345A3A1B9D53D37DC64646B16
answer VNET_1_HOSTONLY_NETMASK 255.255.255.0
answer VNET_1_HOSTONLY_SUBNET 192.168.221.0
answer VNET_1_VIRTUAL_ADAPTER yes
answer VNET_8_DHCP yes
answer VNET_8_DHCP_CFG_HASH B871CF5BA10C8B875E2D9E45A46855B9E8644E55
answer VNET_8_HOSTONLY_NETMASK 255.255.255.0
answer VNET_8_HOSTONLY_SUBNET 192.168.240.0
answer VNET_8_NAT yes
answer VNET_8_VIRTUAL_ADAPTER yes
add_bridge_mapping en1 212345678910111213
```

把上面的DHCP 该成NO就行了。 
**注意：至于不知道是修改那一个的，直接看又没有NAT的，没有头NAT的那个就是专用网络，然后修改保存后，必须右键完全退出VM以后重新打开才会生效**
