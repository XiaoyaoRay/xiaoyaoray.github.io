---
title: Centos7纯手工安装kubernetes
copyright: true
date: 2018-08-14 20:41:04
categories:
- Kubernetes
tags:
- Kubernetes
---

## 简介

> 本文章主要介绍如何通过使用官方提供的二进制包安装配置k8s集群

## 实验环境说明

> VMWare上安装三台虚拟机

```
ray01: master 172.16.242.129
ray02: node 172.16.242.130
ray03: node 172.16.242.131
```

<!--more-->

## 安装

### 软件安装前准备工作

> 需要在所有节点上做如下操作

#### 关闭防火墙

> 如果各个主机启用了防火墙，需要开放Kubernetes各个组件所需要的端口，可以查看[Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)中的”Check required ports”一节。 这里简单起见在各节点禁用防火墙：

```
systemctl stop firewalld
systemctl disable firewalld
```

#### 禁用SELINUX

```
# 临时禁用
setenforce 0

# 永久禁用 SELINUX字段改为disabled
vim /etc/selinux/config	# 或者修改/etc/sysconfig/selinux
SELINUX=disabled
```

#### 修改k8s.conf文件

```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness = 0
EOF
sysctl --system
```

#### 关闭swap

```
# 临时关闭
swapoff -a
```

> 修改 /etc/fstab 文件，注释掉 SWAP 的自动挂载（永久关闭swap，重启后生效）

```
# 注释掉以下字段
/dev/mapper/cl-swap     swap                    swap    defaults        0 0
```

### 配置hosts解析

> 如下操作在所有节点操作

```
cat >/etc/hosts<<EOF
172.16.242.129 ray01
172.16.242.130 ray02
172.16.242.131 ray03
EOF
```

### 配置免密登录

> 如下操作在所有节点操作

```
# 以Master节点ray01为例
ssh-copy-id -i ~/.ssh/id_rsa.pub root@ray02
ssh-copy-id -i ~/.ssh/id_rsa.pub root@rah03
```

### 安装Docker

> 每个节点均要进行操作

#### 卸载老版本的Docker

> 如果有没有老版本Docker，则不需要这步

```
yum remove docker \
           docker-common \
           docker-selinux \
           docker-engine
```

#### 使用yum进行安装

> v1.11.0版本推荐使用docker v17.03, v1.11,v1.12,v1.13, 也可以使用，再高版本的docker可能无法正常使用。 测试发现17.09无法正常使用，不能使用资源限制(内存CPU)。每个节点均要安装，目前官网建议安装17.03版本的docker，[官网链接](https://kubernetes.io/docs/setup/independent/install-kubeadm/)

```
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce docker-ce-selinux
# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ce.repo
#   将 [docker-ce-test] 下方的 enabled=0 修改为 enabled=1
#
# 安装指定版本的Docker-CE:
# Step 3.1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step 3.2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)
sudo yum -y --setopt=obsoletes=0 install docker-ce-[VERSION] \
docker-ce-selinux-[VERSION]

# Step 4: 开启Docker服务
sudo systemctl enable docker && systemctl start docker
```

#### docker安装问题小结

> 错误信息

```
Error: Package: docker-ce-17.03.2.ce-1.el7.centos.x86_64 (docker-ce-stable)
           Requires: docker-ce-selinux >= 17.03.2.ce-1.el7.centos
           Available: docker-ce-selinux-17.03.0.ce-1.el7.centos.noarch (docker-ce-stable)
               docker-ce-selinux = 17.03.0.ce-1.el7.centos
           Available: docker-ce-selinux-17.03.1.ce-1.el7.centos.noarch (docker-ce-stable)
               docker-ce-selinux = 17.03.1.ce-1.el7.centos
           Available: docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch (docker-ce-stable)
               docker-ce-selinux = 17.03.2.ce-1.el7.centos
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
```

> 解决办法

```
# 要先安装docker-ce-selinux-17.03.2.ce，否则安装docker-ce会报错
# 注意docker-ce-selinux的版本 要与docker的版本一直
yum install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm

# 或者
yum -y install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm
```

> 参考链接

https://blog.csdn.net/csdn_duomaomao/article/details/79019764

#### Docker安装校验

```
docker version
Client:
 Version:      17.03.2-ce
 API version:  1.27
 Go version:   go1.7.5
 Git commit:   f5ec1e2
 Built:        Tue Jun 27 02:21:36 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.03.2-ce
 API version:  1.27 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   f5ec1e2
 Built:        Tue Jun 27 02:21:36 2017
 OS/Arch:      linux/amd64
 Experimental: false
```

#### 参考文献

https://yq.aliyun.com/articles/110806

### 安装CFSSL

> 只在Master节点`ray01`节点操作

```
# 下载
# 百度云链接：https://pan.baidu.com/s/1kgV40nwHy1IKnnLD6zH4cQ 密码：alyj
mkdir -pv /server/software/k8s
cd /server/software/k8s
yum install -y wget
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64

# 安装
mv cfssl-certinfo_linux-amd64 /usr/local/bin/cfssl-certinfo
mv cfssl_linux-amd64 /usr/local/bin/cfssl
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
chmod +x /usr/local/bin/cfssl*
```

### 配置CA

> 只在Master节点`ray01`节点操作
>
> 此处的CA配置，后面配置etcd和k8s时都需要使用

```
mkdir -pv $HOME/ssl && cd $HOME/ssl
cat >ca-config.json<<EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF
```

### 配置etcd集群

#### 生成etcd-ca

> 只在Master节点`ray01`节点操作

```
# 写入配置
cat >etcd-ca-csr.json<<EOF
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "etcd",
      "OU": "Etcd Security"
    }
  ]
}
EOF

# 生成 etcd root ca
cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare etcd-ca

cat >etcd-csr.json<<EOF
{
    "CN": "etcd",
    "hosts": [
      "127.0.0.1",
      "172.16.242.129",
      "172.16.242.130",
      "172.16.242.131"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "etcd",
            "OU": "Etcd Security"
        }
    ]
}
EOF

# 生成 etcd ca
cfssl gencert -ca=etcd-ca.pem -ca-key=etcd-ca-key.pem -config=ca-config.json \
-profile=kubernetes etcd-csr.json | cfssljson -bare etcd
mkdir -pv /etc/etcd/ssl
cp etcd*.pem /etc/etcd/ssl
ls /etc/etcd/ssl/etcd*.pem

# 复制到其他节点
cd /etc/etcd && tar cvzf etcd-ssl.tgz ssl/
scp /etc/etcd/etcd-ssl.tgz ray02:~/
scp /etc/etcd/etcd-ssl.tgz ray03:~/
ssh ray02 'mkdir -pv /etc/etcd && tar xf etcd-ssl.tgz -C /etc/etcd && ls -l /etc/etcd/ssl'
ssh ray03 'mkdir -pv /etc/etcd && tar xf etcd-ssl.tgz -C /etc/etcd && ls -l /etc/etcd/ssl'
```

#### 安装启动etcd

> 如下操作在所有节点操作

```
# 安装
# 百度云链接：https://pan.baidu.com/s/1IVHyMqiJrlq9gmbF49Ly3Q 密码：w5nx
mkdir -pv /server/software/k8s
cd /server/software/k8s
yum install -y wget
wget https://github.com/coreos/etcd/releases/download/v3.2.18/etcd-v3.2.18-linux-amd64.tar.gz

# 注意：可以用scp拷贝etcd安装包到其他节点
# scp etcd-v3.2.18-linux-amd64.tar.gz ray02:/server/software/k8s
# scp etcd-v3.2.18-linux-amd64.tar.gz ray03:/server/software/k8s

tar xf etcd-v3.2.18-linux-amd64.tar.gz
mv etcd-v3.2.18-linux-amd64 /usr/local/etcd-v3.2.18
ln -sv /usr/local/etcd-v3.2.18 /usr/local/etcd
cd /usr/local/etcd && mkdir bin && mv etcd etcdctl bin
/usr/local/etcd/bin/etcd --version
cd $HOME

# 配置启动脚本
export ETCD_NAME=$(hostname)
export INTERNAL_IP=$(hostname -i | awk '{print $NF}')
export ECTD_CLUSTER='ray01=https://172.16.242.129:2380,ray02=https://172.16.242.130:2380,ray03=https://172.16.242.131:2380'
mkdir -pv /data/etcd
cat > /etc/systemd/system/etcd.service <<EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/data/etcd
EnvironmentFile=-/etc/etcd/etcd.conf
ExecStart=/usr/local/etcd/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/ssl/etcd.pem \\
  --key-file=/etc/etcd/ssl/etcd-key.pem \\
  --peer-cert-file=/etc/etcd/ssl/etcd.pem \\
  --peer-key-file=/etc/etcd/ssl/etcd-key.pem \\
  --trusted-ca-file=/etc/etcd/ssl/etcd-ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ssl/etcd-ca.pem \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token my-etcd-token \\
  --initial-cluster $ECTD_CLUSTER \\
  --initial-cluster-state new \\
  --data-dir=/data/etcd
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# 启动并设置开机启动
systemctl daemon-reload
systemctl start etcd
systemctl enable etcd
```

#### 查看etcd集群状态

> 可以在每个节点上执行以下命令

```
# 执行命令
/usr/local/etcd/bin/etcdctl --endpoints "https://127.0.0.1:2379" \
  --ca-file=/etc/etcd/ssl/etcd-ca.pem \
  --cert-file=/etc/etcd/ssl/etcd.pem \
  --key-file=/etc/etcd/ssl/etcd-key.pem \
  cluster-health
  
# 执行结果
member 2348823a457a26 is healthy: got healthy result from https://172.16.242.131:2379
member 3a4edb8ea263762b is healthy: got healthy result from https://172.16.242.130:2379
member c8530ac4a96f8c86 is healthy: got healthy result from https://172.16.242.129:2379
cluster is healthy
```

### 生成k8s集群的CA

> 只需在Master节点`ray01`上操作，最后把CA拷贝到其他节点（`ray02`和`ray03`）

```
# 进入相关目录
cd $HOME/ssl

# 配置 root ca
cat >ca-csr.json<<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ],
  "ca": {
     "expiry": "87600h"
  }
}
EOF

# 生成 root ca
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
ls ca*.pem

# 配置 kube-apiserver ca
# 10.96.0.1 是 kube-apiserver 指定的 service-cluster-ip-range 网段的第一个IP
cat >kube-apiserver-csr.json<<EOF
{
    "CN": "kube-apiserver",
    "hosts": [
      "127.0.0.1",
      "172.16.242.129",
      "172.16.242.130",
      "172.16.242.131",
      "10.96.0.1",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF

# 生成 kube-apiserver ca
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json \
-profile=kubernetes kube-apiserver-csr.json | cfssljson -bare kube-apiserver
ls kube-apiserver*.pem

# 配置 kube-controller-manager ca
cat >kube-controller-manager-csr.json<<EOF
{
    "CN": "system:kube-controller-manager",
    "hosts": [
      "127.0.0.1",
      "172.16.242.129",
      "172.16.242.130",
      "172.16.242.131"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "system:kube-controller-manager",
            "OU": "System"
        }
    ]
}
EOF

# 生成 kube-controller-manager ca
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json \
-profile=kubernetes kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
ls kube-controller-manager*.pem

# 配置 kube-scheduler ca
cat >kube-scheduler-csr.json<<EOF
{
    "CN": "system:kube-scheduler",
    "hosts": [
      "127.0.0.1",
      "172.16.242.129",
      "172.16.242.130",
      "172.16.242.131"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "system:kube-scheduler",
            "OU": "System"
        }
    ]
}
EOF

# 生成 kube-scheduler ca
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json \
-profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler
ls kube-scheduler*.pem

# 配置 kube-proxy ca
cat >kube-proxy-csr.json<<EOF
{
    "CN": "system:kube-proxy",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "system:kube-proxy",
            "OU": "System"
        }
    ]
}
EOF

# 生成 kube-proxy ca
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json \
-profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
ls kube-proxy*.pem

# 配置 admin ca
cat >admin-csr.json<<EOF
{
    "CN": "admin",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "system:masters",
            "OU": "System"
        }
    ]
}
EOF

# 生成 admin ca
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json \
-profile=kubernetes admin-csr.json | cfssljson -bare admin
ls admin*.pem

# 复制生成的ca
mkdir -pv /etc/kubernetes/pki
cp ca*.pem admin*.pem kube-proxy*.pem kube-scheduler*.pem kube-controller-manager*.pem kube-apiserver*.pem /etc/kubernetes/pki
cd /etc/kubernetes && tar cvzf pki.tgz pki/
scp /etc/kubernetes/pki.tgz ray02:~/
scp /etc/kubernetes/pki.tgz ray03:~/
ssh ray02 'mkdir -pv /etc/kubernetes && tar xf pki.tgz -C /etc/kubernetes && ls -l /etc/kubernetes/pki'
ssh ray03 'mkdir -pv /etc/kubernetes && tar xf pki.tgz -C /etc/kubernetes && ls -l /etc/kubernetes/pki'
cd $HOME
```

### 安装k8s文件

#### 在Master节点k8s的server文件

> 在Master节点`ray01`上运行以下命令

```
# 下载文件
# 需要翻墙，如果不能翻墙使用如下链接下载
# 链接：https://pan.baidu.com/s/1OI9Q4BRp7jNJUmsA8IAkbA 密码：tnx5
cd /server/software/k8s
wget https://dl.k8s.io/v1.11.0/kubernetes-server-linux-amd64.tar.gz
tar xf kubernetes-server-linux-amd64.tar.gz
cd kubernetes/server/bin
mkdir -pv /usr/local/kubernetes-v1.11.0/bin
cp kube-apiserver kube-controller-manager kube-scheduler kube-proxy kubelet kubectl /usr/local/kubernetes-v1.11.0/bin
ln -sv /usr/local/kubernetes-v1.11.0 /usr/local/kubernetes
cp /usr/local/kubernetes/bin/kubectl /usr/local/bin/kubectl
kubectl version
cd $HOME
```

#### 在其他node节点安装k8s的node文件

> 在`ray02`和`ray03`节点上运行

```
cd /server/software/k8s
wget https://dl.k8s.io/v1.11.0/kubernetes-node-linux-amd64.tar.gz
tar xf kubernetes-node-linux-amd64.tar.gz
cd kubernetes/server/bin
mkdir -pv /usr/local/kubernetes-v1.11.0/bin
cp kube-proxy kubelet kubectl /usr/local/kubernetes-v1.11.0/bin
ln -sv /usr/local/kubernetes-v1.11.0 /usr/local/kubernetes
cp /usr/local/kubernetes/bin/kubectl /usr/local/bin/kubectl
kubectl version
cd $HOME
```

### 生成kubeconfig

> 在Master节点`ray01`上运行，最后把 kube-proxy.conf 复制到其他节点（`ray02`和`ray03`）

```
# 使用 TLS Bootstrapping 
export BOOTSTRAP_TOKEN=$(head -c 16 /dev/urandom | od -An -t x | tr -d ' ')
cat > /etc/kubernetes/token.csv <<EOF
${BOOTSTRAP_TOKEN},kubelet-bootstrap,10001,"system:kubelet-bootstrap"
EOF

# 创建 kubelet bootstrapping kubeconfig
cd /etc/kubernetes
export KUBE_APISERVER="https://172.16.242.129:6443"
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kubelet-bootstrap.conf
kubectl config set-credentials kubelet-bootstrap \
  --token=${BOOTSTRAP_TOKEN} \
  --kubeconfig=kubelet-bootstrap.conf
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kubelet-bootstrap \
  --kubeconfig=kubelet-bootstrap.conf
kubectl config use-context default --kubeconfig=kubelet-bootstrap.conf

# 创建 kube-controller-manager kubeconfig
export KUBE_APISERVER="https://172.16.242.129:6443"
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-controller-manager.conf
kubectl config set-credentials kube-controller-manager \
  --client-certificate=/etc/kubernetes/pki/kube-controller-manager.pem \
  --client-key=/etc/kubernetes/pki/kube-controller-manager-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-controller-manager.conf
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-controller-manager \
  --kubeconfig=kube-controller-manager.conf
kubectl config use-context default --kubeconfig=kube-controller-manager.conf

# 创建 kube-scheduler kubeconfig
export KUBE_APISERVER="https://172.16.242.129:6443"
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-scheduler.conf
kubectl config set-credentials kube-scheduler \
  --client-certificate=/etc/kubernetes/pki/kube-scheduler.pem \
  --client-key=/etc/kubernetes/pki/kube-scheduler-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-scheduler.conf
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-scheduler \
  --kubeconfig=kube-scheduler.conf
kubectl config use-context default --kubeconfig=kube-scheduler.conf

# 创建 kube-proxy kubeconfig
export KUBE_APISERVER="https://172.16.242.129:6443"
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-proxy.conf
kubectl config set-credentials kube-proxy \
  --client-certificate=/etc/kubernetes/pki/kube-proxy.pem \
  --client-key=/etc/kubernetes/pki/kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.conf
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.conf
kubectl config use-context default --kubeconfig=kube-proxy.conf

# 创建 admin kubeconfig
export KUBE_APISERVER="https://172.16.242.129:6443"
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=admin.conf
kubectl config set-credentials admin \
  --client-certificate=/etc/kubernetes/pki/admin.pem \
  --client-key=/etc/kubernetes/pki/admin-key.pem \
  --embed-certs=true \
  --kubeconfig=admin.conf
kubectl config set-context default \
  --cluster=kubernetes \
  --user=admin \
  --kubeconfig=admin.conf
kubectl config use-context default --kubeconfig=admin.conf

# 把 kube-proxy.conf 复制到其他节点
scp kubelet-bootstrap.conf kube-proxy.conf ray02:/etc/kubernetes
scp kubelet-bootstrap.conf kube-proxy.conf ray03:/etc/kubernetes
cd $HOME
```

### 配置Master相关组件

> 在Master`ray01`节点操作

#### 配置启动kube-apiserver

> 在Master`ray01`节点操作

```
# 复制 etcd ca
mkdir -pv /etc/kubernetes/pki/etcd
cd /etc/etcd/ssl
cp etcd-ca.pem etcd-key.pem etcd.pem /etc/kubernetes/pki/etcd

# 生成 service account key
openssl genrsa -out /etc/kubernetes/pki/sa.key 2048
openssl rsa -in /etc/kubernetes/pki/sa.key -pubout -out /etc/kubernetes/pki/sa.pub
ls /etc/kubernetes/pki/sa.*
cd $HOME

# 启动文件
cat >/etc/systemd/system/kube-apiserver.service<<EOF
[Unit]
Description=Kubernetes API Service
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/apiserver
ExecStart=/usr/local/kubernetes/bin/kube-apiserver \\
	    \$KUBE_LOGTOSTDERR \\
	    \$KUBE_LOG_LEVEL \\
	    \$KUBE_ETCD_ARGS \\
	    \$KUBE_API_ADDRESS \\
	    \$KUBE_SERVICE_ADDRESSES \\
	    \$KUBE_ADMISSION_CONTROL \\
	    \$KUBE_APISERVER_ARGS
Restart=on-failure
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# 该配置文件同时被 kube-apiserver, kube-controller-manager
# kube-scheduler, kubelet, kube-proxy 使用
cat >/etc/kubernetes/config<<EOF
KUBE_LOGTOSTDERR="--logtostderr=true"
KUBE_LOG_LEVEL="--v=2"
EOF

cat >/etc/kubernetes/apiserver<<EOF
KUBE_API_ADDRESS="--advertise-address=172.16.242.129"
KUBE_ETCD_ARGS="--etcd-servers=https://172.16.242.129:2379,https://172.16.242.130:2379,https://172.16.242.131:2379 --etcd-cafile=/etc/kubernetes/pki/etcd/etcd-ca.pem --etcd-certfile=/etc/kubernetes/pki/etcd/etcd.pem --etcd-keyfile=/etc/kubernetes/pki/etcd/etcd-key.pem"
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.96.0.0/12"
KUBE_ADMISSION_CONTROL="--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota"
KUBE_APISERVER_ARGS="--allow-privileged=true --authorization-mode=Node,RBAC --enable-bootstrap-token-auth=true --token-auth-file=/etc/kubernetes/token.csv --service-node-port-range=30000-32767 --tls-cert-file=/etc/kubernetes/pki/kube-apiserver.pem --tls-private-key-file=/etc/kubernetes/pki/kube-apiserver-key.pem --client-ca-file=/etc/kubernetes/pki/ca.pem --service-account-key-file=/etc/kubernetes/pki/sa.pub --enable-swagger-ui=true --secure-port=6443 --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --anonymous-auth=false --kubelet-client-certificate=/etc/kubernetes/pki/admin.pem --kubelet-client-key=/etc/kubernetes/pki/admin-key.pem"
EOF

# 启动
systemctl daemon-reload
systemctl enable kube-apiserver
systemctl start kube-apiserver
systemctl status kube-apiserver

# 浏览器访问测试
https://172.16.242.129:6443/swaggerapi
```

#### 配置启动kube-controller-manager

> 在Master`ray01`节点操作

```
# 启动文件
cat >/etc/systemd/system/kube-controller-manager.service<<EOF
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/controller-manager
ExecStart=/usr/local/kubernetes/bin/kube-controller-manager \\
	    \$KUBE_LOGTOSTDERR \\
	    \$KUBE_LOG_LEVEL \\
	    \$KUBECONFIG \\
	    \$KUBE_CONTROLLER_MANAGER_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

cat >/etc/kubernetes/controller-manager<<EOF
KUBECONFIG="--kubeconfig=/etc/kubernetes/kube-controller-manager.conf"
KUBE_CONTROLLER_MANAGER_ARGS="--address=127.0.0.1 --cluster-cidr=10.244.0.0/16 --cluster-name=kubernetes --cluster-signing-cert-file=/etc/kubernetes/pki/ca.pem --cluster-signing-key-file=/etc/kubernetes/pki/ca-key.pem --service-account-private-key-file=/etc/kubernetes/pki/sa.key --root-ca-file=/etc/kubernetes/pki/ca.pem --leader-elect=true --use-service-account-credentials=true --node-monitor-grace-period=10s --pod-eviction-timeout=10s --allocate-node-cidrs=true --controllers=*,bootstrapsigner,tokencleaner"
EOF

# 启动
systemctl daemon-reload
systemctl enable kube-controller-manager
systemctl start kube-controller-manager
systemctl status kube-controller-manager
```

#### 配置启动kube-scheduler

> 在Master`ray01`节点操作

```
cat >/etc/systemd/system/kube-scheduler.service<<EOF
[Unit]
Description=Kubernetes Scheduler Plugin
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/scheduler
ExecStart=/usr/local/kubernetes/bin/kube-scheduler \\
            \$KUBE_LOGTOSTDERR \\
            \$KUBE_LOG_LEVEL \\
            \$KUBECONFIG \\
            \$KUBE_SCHEDULER_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

cat >/etc/kubernetes/scheduler<<EOF
KUBECONFIG="--kubeconfig=/etc/kubernetes/kube-scheduler.conf"
KUBE_SCHEDULER_ARGS="--leader-elect=true --address=127.0.0.1"
EOF

# 启动
systemctl daemon-reload
systemctl enable kube-scheduler
systemctl start kube-scheduler
systemctl status kube-scheduler
```

#### 配置kubectl使用

> 在Master`ray01`节点操作

```
rm -rf $HOME/.kube
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
```

##### 查看组件状态

> 在Master`ray01`节点操作

```
kubectl get componentstatuses

# 运行结果
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-1               Healthy   {"health": "true"}
etcd-2               Healthy   {"health": "true"}
etcd-0               Healthy   {"health": "true"}
```

#### 配置kubelet使用bootstrap

> 在Master`ray01`节点操作

```
# 将 bootstrap token 文件中的 kubelet-bootstrap 用户赋予 system:node-bootstrapper cluster 角色
kubectl create clusterrolebinding kubelet-bootstrap \
--clusterrole=system:node-bootstrapper \
--user=kubelet-bootstrap
```

### 配置node相关组件

> 如下操作在所有节点操作

#### 安装cni

> 如下操作在所有节点操作

```
# 安装 cni
# 百度云链接：https://pan.baidu.com/s/1-PputObLs5jouXLnuBCI6Q 密码：tzqm
cd /server/software/k8s
wget https://github.com/containernetworking/plugins/releases/download/v0.7.1/cni-plugins-amd64-v0.7.1.tgz

# 注意
# 下载完成后可以scp到其他节点
# scp cni-plugins-amd64-v0.7.1.tgz ray02:/server/software/k8s
# scp cni-plugins-amd64-v0.7.1.tgz ray03:/server/software/k8s

mkdir -pv /opt/cni/bin
tar xf cni-plugins-amd64-v0.7.1.tgz -C /opt/cni/bin
ls -l /opt/cni/bin
cd $HOME
```

#### 配置启动kubelet

> 如下操作在所有节点操作

```
# 启动文件
mkdir -pv /data/kubelet
cat >/etc/systemd/system/kubelet.service<<EOF
[Unit]
Description=Kubernetes Kubelet Server
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
WorkingDirectory=/data/kubelet
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/kubelet
ExecStart=/usr/local/kubernetes/bin/kubelet \\
            \$KUBE_LOGTOSTDERR \\
            \$KUBE_LOG_LEVEL \\
            \$KUBELET_CONFIG \\
            \$KUBELET_HOSTNAME \\
            \$KUBELET_POD_INFRA_CONTAINER \\
            \$KUBELET_ARGS
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

cat >/etc/kubernetes/config<<EOF
KUBE_LOGTOSTDERR="--logtostderr=true"
KUBE_LOG_LEVEL="--v=2"
EOF

# 注意修改相关ip
cat >/etc/kubernetes/kubelet<<EOF
KUBELET_HOSTNAME="--hostname-override=172.16.242.129"
KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1"
KUBELET_CONFIG="--config=/etc/kubernetes/kubelet-config.yml"
KUBELET_ARGS="--bootstrap-kubeconfig=/etc/kubernetes/kubelet-bootstrap.conf --kubeconfig=/etc/kubernetes/kubelet.conf --cert-dir=/etc/kubernetes/pki --network-plugin=cni --cni-bin-dir=/opt/cni/bin --cni-conf-dir=/etc/cni/net.d"
EOF

# 注意修改相关ip
# ray01 ray02 ray03 使用各自ip
cat >/etc/kubernetes/kubelet-config.yml<<EOF
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 172.16.242.129
port: 10250
cgroupDriver: cgroupfs
clusterDNS:
  - 10.96.0.10
clusterDomain: cluster.local.
hairpinMode: promiscuous-bridge
serializeImagePulls: false
authentication:
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.pem
EOF

# 启动
systemctl daemon-reload
systemctl enable kubelet
systemctl start kubelet
systemctl status kubelet
```

#### 通过证书请求

> 如下操作在所有节点操作，即在配置了`kubectl`的节点上执行如下操作

```
# 查看
kubectl get csr

# 通过
kubectl certificate approve node-csr-Yiiv675wUCvQl3HH11jDr0cC9p3kbrXWrxvG3EjWGoE

# 查看节点
# 此时节点状态为 NotReady
kubectl get nodes

# 在node节点查看生成的文件
ls -l /etc/kubernetes/kubelet.conf
ls -l /etc/kubernetes/pki/kubelet*
```

#### 配置启动kube-proxy

> 如下操作在所有节点操作

```
# 安装
yum install -y conntrack-tools

# 启动文件
cat >/etc/systemd/system/kube-proxy.service<<EOF
[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/proxy
ExecStart=/usr/local/kubernetes/bin/kube-proxy \\
	    \$KUBE_LOGTOSTDERR \\
	    \$KUBE_LOG_LEVEL \\
	    \$KUBECONFIG \\
	    \$KUBE_PROXY_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# 注意修改相关ip
# ray01 ray02 ray03 使用各自ip
# 由于 1.11.0 ipvs 在centos7上有bug无法正常使用
# 实验使用 iptables 模式
# 以后版本可以使用 ipvs 模式
cat >/etc/kubernetes/proxy<<EOF
KUBECONFIG="--kubeconfig=/etc/kubernetes/kube-proxy.conf"
KUBE_PROXY_ARGS="--bind-address=172.16.242.129 --proxy-mode=iptables --hostname-override=172.16.242.129 --cluster-cidr=10.244.0.0/16"
EOF

# 启动
systemctl daemon-reload
systemctl enable kube-proxy
systemctl start kube-proxy
systemctl status kube-proxy
```

### 设置集群角色

> 在Master节点`ray01`运行以下命令

```
# 设置 ray01 为 master
kubectl label nodes 172.16.242.129 node-role.kubernetes.io/master=

# 设置 ray02 ray03 为 node
kubectl label nodes 172.16.242.130 node-role.kubernetes.io/node=
kubectl label nodes 172.16.242.131 node-role.kubernetes.io/node=

# 设置 master 一般情况下不接受负载
kubectl taint nodes 172.16.242.129 node-role.kubernetes.io/master=true:NoSchedule

# 查看节点
# 此时节点状态为 NotReady
kubectl get no
```

### 配置使用flannel网络

> 在Master节点`ray01`操作

```
# 下载配置
mkdir flannel && cd flannel
wget https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml

# 修改配置
# 此处的ip配置要与上面kubeadm的pod-network一致
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }

# 修改镜像
image: registry.cn-shanghai.aliyuncs.com/gcr-k8s/flannel:v0.10.0-amd64

# 如果Node有多个网卡的话，参考flannel issues 39701，
# https://github.com/kubernetes/kubernetes/issues/39701
# 目前需要在kube-flannel.yml中使用--iface参数指定集群主机内网网卡的名称，
# 否则可能会出现dns无法解析。容器无法通信的情况，需要将kube-flannel.yml下载到本地，
# flanneld启动参数加上--iface=<iface-name>
    containers:
      - name: kube-flannel
        image: registry.cn-shanghai.aliyuncs.com/gcr-k8s/flannel:v0.10.0-amd64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=eth1

# 启动
kubectl apply -f kube-flannel.yml

# 查看
kubectl get pods -n kube-system
kubectl get svc -n kube-system

# 查看节点状态
# 当 flannel pod 全部启动之后，节点状态为 Ready
kubectl get no

# 运行结果
NAME             STATUS    ROLES     AGE       VERSION
172.16.242.129   Ready     master    5h        v1.11.0
172.16.242.130   Ready     node      4h        v1.11.0
172.16.242.131   Ready     node      4h        v1.11.0
```

### 配置使用coredns

> 在Master节点`ray01`操作

```
# 安装
# 10.96.0.10 kubelet中配置的dns
cd $HOME && mkdir coredns && cd coredns
wget https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/coredns.yaml.sed
wget https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/deploy.sh
chmod +x deploy.sh
./deploy.sh -i 10.96.0.10 > coredns.yml
kubectl apply -f coredns.yml

# 查看
kubectl get pods -n kube-system
kubectl get svc -n kube-system
```

## 测试

### 启动

```
kubectl run nginx --replicas=2 --image=nginx:alpine --port=80
kubectl expose deployment nginx --type=NodePort --name=example-service-nodeport
kubectl expose deployment nginx --name=example-service
kubectl scale --replicas=3 deployment/nginx
```

### 查看状态

```
kubectl get deploy -o wide
kubectl get pods -o wide
kubectl get svc -o wide
kubectl describe svc example-service
```

### DNS解析

```
kubectl run curl --image=radial/busyboxplus:curl -i --tty
nslookup kubernetes
nslookup example-service
curl example-service
```

### 访问测试

```
# 10.96.59.56 为查看svc时获取到的clusterip
curl "10.107.91.153:80"

# 32223 为查看svc时获取到的 nodeport
http://172.16.242.129:32223/
http://172.16.242.130:32223/
http://172.16.242.131:32223/
```

### 清理

```
kubectl delete svc example-service example-service-nodeport
kubectl delete deploy nginx curl
```

 ## 参考

[centos7纯手动安装kubernetes-v1.11版本](https://juejin.im/post/5b45cea9f265da0f652370ce#heading-25)
