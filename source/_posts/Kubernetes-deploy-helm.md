---
title: kubernetes-helm部署及本地repo搭建
date: 2018-12-26 11:51:22
categories:
- Kubernetes
tags:
- Kubernetes
- Helm
---

# 转载：[kubernetes-helm部署及本地repo搭建](https://blog.csdn.net/liukuan73/article/details/79319900)

### Helm架构

![20180215101704391](https://ws3.sinaimg.cn/large/006tNbRwgy1fyk3osca4oj30v20f4nht.jpg)

<!--more-->

主要由3部分组成：

- helm client 
  - 用来部署Tiller server
  - 用来管理Chart repository
  - 用来管理Chart package
- tiller 
  - 用来管理release
  - chart repo

### Helm Client安装
helm client主要作用如下：

  - 用来部署Tiller server
  - 用来管理Chart repository
  - 用来管理Chart package

到helm的release页面下载最新helm二进制包安装：

```
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-linux-amd64.tar.gz
tar -zxvf helm-v2.9.1-linux-amd64.tar.gz
cd linux-amd64
mv helm /usr/local/bin/
```



### Helm Tiller安装
Helm Tiller是Helm的server，用来管理release。

Tiller有多种安装方式，比如本地安装或以pod形式部署到Kubernetes集群中。本文以pod安装为例，安装Tiller的最简单方式是helm init, 该命令会检查helm本地环境设置是否正确，helm init会连接kubectl默认连接的kubernetes集群（可以通过kubectl config view查看），一旦连接集群成功，tiller会被安装到kube-system namespace中。

#### 执行helm init 
该命令会通过Kubernetes的Deployment 部署tiller，并且在本机配置一个名为local的本地repo，当前用户目录下会生成.helm文件夹（即~/.helm）放置repo相关内容。

helm init主要做了以下三件事情：

- 部署Tiller

- 初始化本地 cache

- 初始化本地 chart仓库

**备注：** 

- 为了起在指定节点上和使用已有镜像，我修改了tiller的deployment：

  ```
    image: daocloud.io/liukuan73/tiller-lk:v2.9.1
    nodeSelector:
      node-role.kubernetes.io/master: "true"
    tolerations:
    - key: node-role.kubernetes.io/master
      effect: NoSchedule
  ```



- 修改tiller的svc使其暴露nodePort端口： 
    kubectl edit svc tiller-deploy -n kube-system

    ```
      ports:
      - name: tiller
        nodePort: 32134
      type: NodePort
    ```

- Tiller还可以通过指定启动参数的形式修改这些配置：

    ```
    安装金丝雀build： --canary-image
    安装指定image：--tiller-image
    指定某一个Kubernetes集群：--kube-context
    指定namespace安装：--tiller-namespace
    
    helm init --tiller-image=daocloud.io/liukuan73/tiller-lk:v2.9.1 --tiller-namespace=kube-system
    ```


- Helm TILLER删除 
    由于 Tiller的数据存储于Kubernetes ConfigMap中，所以删除、升降级Tiller，原Helm部署的应用数据并不会丢失。 删除Tiller：

    ```
    helm reset
    ```



### chart repo
chart repo是一个可用来存储index.yml与打包的chart文件的HTTP server。

当要分享chart时，需要上传chart文件到chart仓库。任何一个能能够提供YAML与tar文件的HTTP server都可以当做chart仓库，比如Google Cloud Storage (GCS) bucket、Amazon S3 bucket、Github Pages或创建你自己的web服务器。官方chart仓库由Kubernetes Charts维护， Helm允许我们创建私有chart仓库。

#### chart repo结构
查看目前的repo:

```
helm repo list

NAME            URL
stable          https://kubernetes-charts.storage.googleapis.com
local           http://127.0.0.1:8879/charts
```


helm执行tiller命令后默认会配置一个名为local的本地repo。

一个chart仓库由一个chart包与index.yaml文件组成，index.yaml记录了chart仓库中全部chart的索引，一个本地chart仓库的布局例子如下：

```
~/.helm/
|-- cache
| `-- archive
| |-- drupal-0.9.2.tgz
| `-- mariadb-1.0.3.tgz
|-- plugins
|-- repository
| |-- cache
| | |-- fantastic-charts-index.yaml
| | |-- local-index.yaml -> /home/ts1/.helm/repository/local/index.yaml
| | |-- mariadb-1.0.3.tgz-index.yaml
| | |-- memcached-1.2.1.tgz-index.yaml
| | |-- mychart_xia-0.1.0.tgz-index.yaml
| | |-- mysql-0.2.8.tgz-index.yaml
| | |-- stable-index.yaml
| | |-- test-0.1.0.tgz-index.yaml
| | `-- test-0.1.8.tgz-index.yaml
| |-- local
| | |-- index.yaml
| | |-- mychart-0.1.0.tgz
| | |-- mychart_xia-0.1.0.tgz
| | |-- mysql-0.2.8.tgz
| | |-- mysql-6.19.centos-29.tgz
| | |-- test-0.1.0.tgz
| | |-- test-0.1.8.tgz
| | `-- test-0.1.9.tgz
| `-- repositories.yaml
`-- starters
```


~/.helm/repository/local/index.yaml文件中记录了chart的诸如名称、url、version等一些metadata信息。

#### 启动repo服务

##### 创建repo目录：

```
mkdir -p /dcos/appstore/local-repo
```

备注： 
此处我没用默认的local repo，而是创建一个新的文件夹并创建一个新的repo演示。 

##### 启动本地repo仓库服务:

```
nohup helm serve --address 0.0.0.0:8879 --repo-path /dcos/appstore/local-repo &
```


##### 通过helm repo add命令添加本地repo：

```
helm repo add local-repo http://10.142.21.21:8879
    "local-repo" has been added to your repositories
```

##### 查看本地chart仓库是否添加成功：

```
helm repo list:

NAME            URL
stable          https://kubernetes-charts.storage.googleapis.com
local           http://127.0.0.1:8879/charts
local-repo  http://10.142.21.21:8879
```


**备注：**

- `helm serve` 不指定任何参数的话会在默认的repo目录（/root/.helm/repository/local 
  ）启动服务，根据该目录下的软件包（tgz）信息在该目录下创建index.html文件。
- 可以通过指定`--repo-path`参数实现在自定义的目录下启动服务，并在那个目录下创建index.html文件。
### 向repo中增加软件包

 上面步骤中，已经创建了一个本地的repo，接下来讲述如何在repo中增加一个可用来部署的软件包chart。chart须遵循 SemVer 2 规则填写正确的版本格式。各种chart包可以在[github](https://github.com/kubernetes/charts)下载。

#### 将chart文件夹移动到repo目录，并将chart打包：

```
cp -r jenkins /dcos/appstore/local-repo/
cd /dcos/appstore/local-repo
helm package jenkins --save=false 
```


**备注:**

- `helm package`的作用是在当前目录下将软件打包为tgz，假如这个软件包中有requirement.yaml，则打包时还需要加上``--dependency-update`，用来`update dependencies from "requirements.yaml" to dir "charts/" before packaging`
- `--save=false`的作用是不将tgz文件再拷贝一份到默认的local chart repo文件夹（`/root/.helm/repository/local/`）下，否则默认会将tgz拷贝一份到那，并检查那个目录下的`index.html`是否存在，不存在会报错。或者在3.2中最后把没用到的local repo删掉，就不用加`--save=false`了。

#### 更新index.yaml文件 
  通过helm repo index 命令将chart的metadata记录更新在index.yaml文件中:

```
cd /dcos/appstore/local-repo
helm repo index --url=http://10.142.21.21:8879 .    
helm repo update
```


**备注： **
这句话的作用是:Read the current directory and generate an index file based on the charts found.

#### 验证

- 查找新上传的chart: 

  ```
  helm search jenkins
  
  NAME                    CHART VERSION   APP VERSION DESCRIPTION
  local-repo/jenkins  0.16.3          2.107       Open source continuous integration server. It s...
  stable/jenkins          0.16.3          2.107       Open source continuous integration server. It s...
  ```

- 安装chart软件包（即release过程）：

  ```
  helm install local-repo/jenkins
  ```


  **备注：**helm install会用到socat，需要在所有节点上安装socat