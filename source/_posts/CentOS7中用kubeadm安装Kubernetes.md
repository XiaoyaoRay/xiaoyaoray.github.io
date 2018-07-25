---
title: CentOS7中用kubeadm安装Kubernetes
copyright: true
tags:
- Kubernetes
---
## 准备

每个节点均要执行以下步骤

### 关闭防火墙

如果各个主机启用了防火墙，需要开放Kubernetes各个组件所需要的端口，可以查看[Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)中的”Check required ports”一节。 这里简单起见在各节点禁用防火墙：

```shell
systemctl stop firewalld
systemctl disable firewalld
```

### 禁用SELINUX

```Shell
# 临时禁用
setenforce 0

# 永久禁用 
vim /etc/selinux/config	# 或者修改/etc/sysconfig/selinux
SELINUX=disabled
```

### 修改k8s.conf文件

```shell
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### 关闭swap

```shell
# 临时关闭
swapoff -a
```

修改 /etc/fstab 文件，注释掉 SWAP 的自动挂载（永久关闭swap，重启后生效）

```shell
# 注释掉以下字段
/dev/mapper/cl-swap     swap                    swap    defaults        0 0
```



## 安装Docker

### 卸载老版本的Docker

如果有没有老版本Docker，则不需要这步

```shell
yum remove docker \
           docker-common \
           docker-selinux \
           docker-engine
```

### 使用yum进行安装

每个节点均要安装，目前官网建议安装17.03版本的docker，[官网链接](https://kubernetes.io/docs/setup/independent/install-kubeadm/)

```Shell
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo systemctl enable docker && systemctl start docker

# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ce.repo
#   将 [docker-ce-test] 下方的 enabled=0 修改为 enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
```

### docker安装问题小结

错误信息

```shell
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

解决办法

```shell
#要先安装docker-ce-selinux-17.03.2.ce，否则安装docker-ce会报错
yum install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm

# 或者
yum -y install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm
```

参考链接

https://blog.csdn.net/csdn_duomaomao/article/details/79019764

### 安装校验

```shell
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

### 参考文献

https://yq.aliyun.com/articles/110806



## 安装kubeadm，kubelet，kubectl

在各节点安装kubeadm，kubelet，kubectl

### 修改yum安装源

```Shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 安装软件

```shell
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```

## 初始化Master节点

### 配置kubeadm init 初始化文件

配置文件[官网介绍链接](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file)

创建配置文件kubeadm-init.yaml文件

```shell
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.1	# kubernetes的版本
api:
  advertiseAddress: 172.16.242.129 # Master的IP地址
networking:
  podSubnet: 192.168.0.0/16	# pod网络的网段
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers # image的仓库源
```

### 运行初始化命令

```shell
kubeadm init --config kubeadm-init.yaml

......
......
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the addon options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

### 使kubectl正常工作

#### 非root用户

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### root用户

```shell
export KUBECONFIG=/etc/kubernetes/admin.conf
```

### 问题解决

[官网解决问题链接](https://kubernetes.io/docs/setup/independent/troubleshooting-kubeadm/)

#### 多次运行kubeadm init命令，需要reset

```shell
kubeadm reset
```

#### cgroup driver报错

```
 error: failed to run Kubelet: failed to create kubelet:
  misconfiguration: kubelet cgroup driver: "systemd" is different from docker cgroup driver: "cgroupfs"
```

需要修改 /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

```
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
systemctl daemon-reload && systemctl restart kubelet
```

#### kubelet不能启动成功原因之一

在`systemctl status docker`中如果出现需要镜像`k8s.gcr.io/pause:3.1`，运行以下命令修改下标签

```shell
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
```



## 安装Pod Network

[官网链接](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network)

接下来安装pod network add-on，以安装Calico为例：

```Shell
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

使用`kubectl get pod --all-namespaces -o wide`确保所有的Pod都处于Running状态。

```Shell
kubectl get pod --all-namespaces -o wide
```

## master node参与工作负载

使用kubeadm初始化的集群，出于安全考虑Pod不会被调度到Master Node上，也就是说Master Node不参与工作负载。

这里搭建的是测试环境可以使用下面的命令使Master Node参与工作负载：

```Shell
kubectl taint nodes node1 node-role.kubernetes.io/master-
node "node1" untainted
```

## 向Kubernetes集群添加Node

[官网链接](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network)

- 添加命令

  ```shell
  kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
  ```

- 查看token的值，在master节点运行以下命令

  ```shell
  # 如果没有token，请使用命令kubeadm token create 创建
  kubeadmin token list
  ```

- 查看hash值，在master节点运行以下命令

  ```shell
  openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
     openssl dgst -sha256 -hex | sed 's/^.* //'
  ```

- node2加入集群很是顺利，下面在master节点上执行命令查看集群中的节点：

  ```shell
  kubectl get nodes
  ```

- 如何从集群中移除Node

  在master节点上执行：

  ```shell
  kubectl drain node2 --delete-local-data --force --ignore-daemonsets
  kubectl delete node <node name>
  ```

  在node2上执行：

  ```shell
  kubeadm reset
  ```



## 附录

### 配置代理
- 配置全局代理
```shell
cat <<EOF >  ~/.bashrc
export http_proxy=http://username:password@ip:port
export https_proxy=http://username:password@ip:port
export no_proxy=localhost,127.0.0.1,<your-server-ip>(本机ip地址)
EOF
source ~/.bashrc
```

- 配置docker代理，拉谷歌镜像要用到
```shell
mkdir -p /etc/systemd/system/docker.service.d/
cat <<EOF > /etc/systemd/system/docker.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://username:password@ip:port" "HTTPS_PROXY=http://username:password@ip:port" "NO_PROXY=localhost,127.0.0.1,<your-server-ip>"
EOF
systemctl daemon-reload
systemctl restart docker
```



## 参考

- [Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)
- [Using kubeadm to Create a Cluster](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
- [Get Docker CE for CentOS](https://docs.docker.com/engine/installation/linux/docker-ce/centos/)
- [Kubernetes Dashboard Access-control](https://github.com/kubernetes/dashboard/wiki/Access-control)
- [使用kubeadm安装Kubernetes 1.9](https://blog.frognew.com/2017/12/kubeadm-install-kubernetes-1.9.html)
