---
title: V2Ray的安装使用
date: 2019-01-25 14:33:56
categories:
- VPN
tags:
- VPN
---

## 安装

### 使用Docker方式安装

- Docker具体安装请见：https://docs.docker.com/install/linux/docker-ce/centos/

- 新建运行目录，比如：`makir v2ray`

- 进入到`v2ray`目录新建一个文件`config.json`，可以在[V2Ray生成配置网站](https://intmainreturn0.com/v2ray-config-gen/#)生成。填写所需的端口，然后复制配置到文件`config.json`

  <!--more-->

  ![image-20190125144154593](https://ws1.sinaimg.cn/large/006tNc79gy1fzitr8y5qxj30m30qatcd.jpg)

- 进入到`v2ray`目录新建一个`Makefile`文件，主要文件中的使用`tab`键

  ```
  .PHONY: run clear
  run: clear
  	docker run -d --privileged=true --name v2ray -v $(PWD):/etc/v2ray --network=host  v2ray/official
  
  clear:
  	-docker rm -f v2ray
  ```

- 在目录`v2ray`中运行命令：`make run`



## 使用

[使用文档](https://www.v2ray.com/ui_client/osx.html)

### 生成客户端配置文件

在[V2Ray生成配置网站](https://intmainreturn0.com/v2ray-config-gen/#)生成，需要填写服务端地址，本地socket使用端口，用户，服务端端口。然后点击复制配置，保存为`client-config.json`

![image-20190125144746186](https://ws1.sinaimg.cn/large/006tNc79gy1fzitxapq80j30d60geq3u.jpg)

### MacOS 连接

- 下载[V2Ray](https://github.com/Cenmrev/V2RayX/releases)，并安装

#### 添加客户端配置文件

- 选择Configure选项

  ![image-20190125145222467](https://ws2.sinaimg.cn/large/006tNc79gy1fziu230odwj30f60luqcf.jpg)

- 选择`Advanced`

  ![image-20190125145508688](https://ws3.sinaimg.cn/large/006tNc79gy1fziu4z270uj30tq0r6grt.jpg)

- 选择`configs`，并添加前面准备好的客户端配置文件，即配置文件的路径（比如：`client-config.json`)

  ![image-20190125145657548](https://ws1.sinaimg.cn/large/006tNc79gy1fziu6us78kj30sc0ogjur.jpg)



#### 启动V2Ray代理

- 选择添加好的服务

  ![image-20190125150217787](https://ws1.sinaimg.cn/large/006tNc79gy1fziuciba5jj30nm0lge43.jpg)

- 选择启动模式，**注意：PAC模式请修改`pac.js`中socket的端口(默认为1081）**

  ![image-20190125150311751](https://ws1.sinaimg.cn/large/006tNc79gy1fziudd1dtsj30f60lqdx5.jpg)

- 启动V2Ray

  ![image-20190125150648887](https://ws3.sinaimg.cn/large/006tNc79gy1fziuh3wbunj30f40lkn9m.jpg)

  

  

  

