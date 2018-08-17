---
title: CentOS安装VPN客户端并连接
date: 2018-08-17 14:10:59
categories:
- Linux
tags:
- Linux
---

## 安装vpn客户端

```
yum -y install ppp pptp pptp-setup
```

## 连接vpn服务

### 创建账号

> --create是创建的连接名称
>
>  --server是vpn的ip地址
>
>  --username是用户名
>
>  --password是密码，也可以没这个参数，命令稍后会自动询问。这样可以保证账号安全
>
>  --encrypt 是表示需要加密，不必指定加密方式，命令会读取配置文件中的加密方式
>
>  --start是表示创建连接完后马上连接

```
# pptpsetup --create vpn --server x.x.x.x --username vpntest --password 1234567890 --encrypt --start
Using interface ppp0
Connect: ppp0 <--> /dev/pts/2
CHAP authentication succeeded
MPPE 128-bit stateless compression enabled
local  IP address 10.0.0.10
remote IP address 10.0.0.1
```

#### 查看创建账号`vpn`的配置文件

```
# written by pptpsetup
# cat /etc/ppp/peers/vpn 
pty "pptp vpn.netkiller.cn --nolaunchpppd"
lock
noauth
nobsdcomp
nodeflate
name neo
remotename vpn
ipparam vpn	
```

#### 内核模块安装

```
for module in nf_nat_pptp nf_conntrack_pptp nf_conntrack_proto_gre
do
    modprobe $module
done	
```

### 拨入VPN

#### 方法一

```
# call 后面添加创建的账号的名称
pppd call vpn
```

#### 方法二

- 先把程序复制到系统路径让系统识别

  ```
  cp /usr/share/doc/ppp-2.4.5/scripts/pon /usr/sbin/ 
  cp /usr/share/doc/ppp-2.4.5/scripts/poff /usr/sbin/ 
  chmod +x /usr/sbin/pon 
  chmod +x /usr/sbin/poff
  ```

- 使用pon开始拨号, poff端开拨号

  ```
  # 连接名称为vpn,vpn服务
  pon vpn
  
  # 断开名称为vpn,vpn服务
  poff vpn
  ```

#### 查看日志

```
# tail -f /var/log/messages | grep pppd
Sep  9 19:09:19 iZ621r6pk9aZ pppd[21801]: pppd 2.4.5 started by root, uid 0
Sep  9 19:09:19 iZ621r6pk9aZ pppd[21801]: Using interface ppp0	
```

### 路由配置

#### 自动配置路由

创建文件/etc/ppp/ip-up.local，写入添加路由命令，然后赋予可执行权限。

```
# cat /etc/ppp/ip-up.local 
ip route add default dev ppp0

# chmod +x /etc/ppp/ip-up.local 
```

创建文件 /etc/ppp/ip-down.local 写入删除路由命令，然后赋予可执行权限

```
# cat /etc/ppp/ip-down.local
ip route del default dev ppp0

chmod +x /etc/ppp/ip-down.local				
```

#### 手工配置路由

添加路由

```
ip route add default dev ppp0		
```

查看路由表

```
# ip route 
default via 47.19.19.27 dev eth1 
1.2.2.2 dev ppp0  proto kernel  scope link  src 2.0.1.8 
10.0.0.0/8 via 10.47.47.247 dev eth0 
10.47.40.0/21 dev eth0  proto kernel  scope link  src 10.47.40.190 
47.89.36.0/22 dev eth1  proto kernel  scope link  src 47.89.36.254 
100.64.0.0/10 via 10.47.47.247 dev eth0 
118.142.17.226 via 47.89.39.247 dev eth1  src 47.89.36.254 
169.254.0.0/16 dev eth0  scope link  metric 1002 
169.254.0.0/16 dev eth1  scope link  metric 1003 
172.16.0.0/12 via 10.47.47.247 dev eth0  
192.168.0.0/24 dev ppp0  scope link			
```

删除路由

```
ip route del default dev ppp0	
```

## FAQ

### centos连接VpnServer 超时,执行下面命令

```
# Delete all rules in  chain or all chains
iptables -F
```

### 连接vpn server，报如下错误

```
Using interface ppp0
Connect: ppp0 <--> /dev/pts/2
EAP: unknown authentication type 26; Naking
EAP: peer reports authentication failure
Connection terminated.
```

解决办法：

```
vi /etc/ppp/options
```

```
将下面字段添加到/etc/ppp/options文件中(root用户权限可更改）

refuse-pap
refuse-eap
refuse-chap
refuse-mschap
require-mppe
```

### 连接成功,路由修改成功，没网

> 错误原因：DNS 没有成功解析

解决办法： 
1.修改/etc/resolv.conf文件，将nameserver 配置成VPN服务器中的dns的ip，或者直接写公用dns（114.114.114.114或1.1.1.1或8.8.8.8）

```
vi /etc/resolv.conf
```

修改如下：

```
nameserver 1.1.1.1
```

### 重启机器或者重启network服务，会断开连接

> 解决方法待续

## 参考

[PPTP VPN 服务器](https://cloud.tencent.com/developer/article/1051293)

[linux连接vpn服务](https://blog.csdn.net/qq_20948497/article/details/53419280)

[Linux centos ***拨号客户端](http://blog.51cto.com/wutou/1737372)