---
title: 使用kubeadm安装Kubernetes1.9
copyright: true
tags:
- Kubernetes
---
kubeadm是Kubernetes官方提供的用于快速安装Kubernetes集群的工具，伴随Kubernetes每个版本的发布都会同步更新，kubeadm会对集群配置方面的一些实践做调整，通过实验kubeadm可以学习到Kubernetes官方在集群配置上一些新的最佳实践。

Kubernetes的官方文档更新的速度太快了，我们注意到在Kubernetes 1.9的文档[Using kubeadm to Create a Cluster](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)中已经给出了目前1.9的kubeadm的主要特性已经处于beta状态了，在2018年将进入GA状态，说明kubeadm离可以在生产环境中使用的距离越来越近了。


## 1.准备

### 1.1 系统配置

如果各个主机启用了防火墙，需要开放Kubernetes各个组件所需要的端口，可以查看[Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)中的”Check required ports”一节。 这里简单起见在各节点禁用防火墙：

```shell
systemctl stop firewalld
systemctl disable firewalld
```

禁用SELINUX：

```Shell
setenforce 0
```

```Shell
vi /etc/selinux/config
SELINUX=disabled
```

创建/etc/sysctl.d/k8s.conf文件，添加如下内容：

```Shell
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

### 1.2安装Docker

卸载老版本的Docker

```shell
yum remove docker \
           docker-common \
           docker-selinux \
           docker-engine
```

设置yum安装的docker-ce.repo

```Shell
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

查看当前的Docker版本：

```Shell
yum list docker-ce.x86_64  --showduplicates |sort -r
docker-ce.x86_64            17.09.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.0.ce-1.el7.centos             docker-ce-stable
```

Kubernetes 1.8已经针对Docker的1.11.2, 1.12.6, 1.13.1和17.03等版本做了验证。 因为我们这里在各节点安装docker的17.03.2版本。

```Shell
yum makecache fast

yum install -y docker-ce-selinux-17.03.2.ce-1.el7.centos

systemctl start docker
systemctl enable docker
```

Docker从1.13版本开始调整了默认的防火墙规则，禁用了iptables filter表中FOWARD链，这样会引起Kubernetes集群中跨Node的Pod无法通信，在各个Docker节点执行下面的命令：

```Shell
iptables -P FORWARD ACCEPT
```

可在docker的systemd unit文件中以ExecStartPost加入上面的命令：

```Shell
ExecStartPost=/usr/sbin/iptables -P FORWARD ACCEPT
```

```Shell
systemctl daemon-reload
systemctl restart docker
```

## 2.安装kubeadm和kubelet

下面在各节点安装kubeadm和kubelet：

```Shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

测试地址https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64是否可用，如果不可用需要科学上网([配置代理](#jump))

```Shell
curl https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
```

```Shell
yum makecache fast
yum install -y kubelet kubeadm kubectl

... 
Installed:
  kubeadm.x86_64 0:1.9.0-0         kubectl.x86_64 0:1.9.0-0           kubelet.x86_64 0:1.9.0-0

Dependency Installed:
  kubernetes-cni.x86_64 0:0.6.0-0      socat.x86_64 0:1.7.3.2-2.el7
```

> - 从安装结果可以看出还安装了kubernetes-cni和socat两个依赖：
>   - 可以看出官方Kubernetes 1.9依赖的cni升级到了0.6.0版本
>   - socat是kubelet的依赖
>   - 我们之前在[Kubernetes 1.6 高可用集群部署](https://blog.frognew.com/2017/04/install-ha-kubernetes-1.6-cluster.html)中手动安装这两个依赖的

Kubernetes文档中kubelet的[启动参数](https://kubernetes.io/docs/admin/kubelet/)：

```
  --cgroup-driver string Driver that the kubelet uses to manipulate cgroups on the host.
		Possible values: 'cgroupfs', 'systemd' (default "cgroupfs")
```

默认值为cgroupfs，但是我们注意到yum安装kubelet,kubeadm时生成10-kubeadm.conf文件中将这个参数值改成了systemd。

查看kubelet的 /etc/systemd/system/kubelet.service.d/10-kubeadm.conf文件，其中包含如下内容：

```Shell
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"
```

使用`docker info`打印docker信息：

```
docker info
......
Server Version: 17.03.2-ce
......
Cgroup Driver: cgroupfs
```

可以看出docker 17.03使用的Cgroup Driver为cgroupfs。

于是修改各节点docker的cgroup driver使其和kubelet一致，即修改或创建/etc/docker/daemon.json，加入下面的内容：

```Shell
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

重启docker:

```Shell
systemctl restart docker
systemctl status docker
```

在各节点开机启动kubelet服务：

```Shell
systemctl enable kubelet.service
```

Kubernetes 1.8开始要求关闭系统的Swap，如果不关闭，默认配置下kubelet将无法启动。可以通过kubelet的启动参数`--fail-swap-on=false`更改这个限制。

> 关闭系统的Swap方法如下:
>
> ```Shell
>   swapoff -a
> ```
>
> 修改 /etc/fstab 文件，注释掉 SWAP 的自动挂载，使用`free -m`确认swap已经关闭。 swappiness参数调整，修改/etc/sysctl.d/k8s.conf添加下面一行：
>
> ```Shell
>   vm.swappiness=0
> ```
>
> 执行`sysctl -p /etc/sysctl.d/k8s.conf`使修改生效。

因为这里本次用于测试两台主机上还运行其他服务，关闭swap可能会对其他服务产生影响，所以这里修改kubelet的启动参数`--fail-swap-on=false`去掉这个限制。修改/etc/systemd/system/kubelet.service.d/10-kubeadm.conf，加入：

```Shell
Environment="KUBELET_EXTRA_ARGS=--fail-swap-on=false"
```

```Shell
systemctl daemon-reload
```

## 3.使用kubeadm init初始化集群

接下来使用kubeadm初始化集群，选择node1作为Master Node，在node1上执行下面的命令：

```Shell
kubeadm init \
  --kubernetes-version=v1.9.0 \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=192.168.61.11
```

因为我们选择flannel作为Pod网络插件，所以上面的命令指定–pod-network-cidr=10.244.0.0/16。

执行时报了下面的错误：

```
[init] Using Kubernetes version: v1.9.0
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks.
        [WARNING FileExisting-crictl]: crictl not found in system path
[preflight] Some fatal errors occurred:
        [ERROR Swap]: running with swap on is not supported. Please disable swap
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
```

一个警告信息是`crictl not found in system path`，另一个错误信息是`running with swap on is not supported. Please disable swap`。因为我们前面已经修改了kubelet的启动参数，所以重新添加–ignore-preflight-errors=Swap参数忽略这个错误，重新运行。

```Shell
kubeadm init \
>   --kubernetes-version=v1.9.0 \
>   --pod-network-cidr=10.244.0.0/16 \
>   --apiserver-advertise-address=192.168.61.11 \
>   --ignore-preflight-errors=Swap
[init] Using Kubernetes version: v1.9.0
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks.
        [WARNING Swap]: running with swap on is not supported. Please disable swap
        [WARNING FileExisting-crictl]: crictl not found in system path
[preflight] Starting the kubelet service
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [node1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.61.11]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "scheduler.conf"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests".
[init] This might take a minute or longer if the control plane images have to be pulled.
[apiclient] All control plane components are healthy after 115.502952 seconds
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[markmaster] Will mark node node1 as master by adding a label and a taint
[markmaster] Master node1 tainted and labelled with key/value: node-role.kubernetes.io/master=""
[bootstraptoken] Using token: 227d9b.9c1772a7ab45942d
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: kube-dns
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 227d9b.9c1772a7ab45942d 192.168.61.11:6443 --discovery-token-ca-cert-hash sha256:e2aa90853cc97410904adc5d58fbb52f4377ad091a0a807bed0dc69e37107151
```

上面记录了完成的初始化输出的内容。我们注意到虽然kubeadm 1.9仍然处于beta状态，但是kubeadm init的输出中已经没有了过去运行kubeadm 1.8时的警告信息`[kubeadm] WARNING: kubeadm is in beta, please do not use it for production clusters.`

> 其中有以下关键内容：
>
> - kubeadm 1.8当前还处于beta状态，还不能用于生产环境。目前来看这东西安装的etcd和apiserver都是单节点，当然不能用于生产环境。
>
> - RBAC模式早已经在Kubernetes 1.8中稳定可用。kubeadm 1.9也默认启用了RBAC
>
> - 接下来是生成证书和相关的kubeconfig文件，这个目前我们在[Kubernetes 1.6 高可用集群部署](https://blog.frognew.com/2017/04/install-ha-kubernetes-1.6-cluster.html)也是这么做的，目前没看出有什么新东西
>
> - 生成token记录下来，后边使用`kubeadm join`往集群中添加节点时会用到
>
> - 下面的命令是配置常规用户如何使用kubectl访问集群：
>
>   ```Shell
>     mkdir -p $HOME/.kube
>     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
>     sudo chown $(id -u):$(id -g) $HOME/.kube/config
>   ```
>
> - 最后给出了将节点加入集群的命令`kubeadm join --token 227d9b.9c1772a7ab45942d 192.168.61.11:6443 --discovery-token-ca-cert-hash sha256:e2aa90853cc97410904adc5d58fbb52f4377ad091a0a807bed0dc69e37107151`

查看一下集群状态：

```Shell
kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health": "true"}
```

确认个组件都处于healthy状态。

集群初始化如果遇到问题，可以使用下面的命令进行清理：

```Shell
kubeadm reset
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/
```

## 4.安装Pod Network

接下来安装flannel network add-on：

```Shell
mkdir -p ~/k8s/
cd ~/k8s
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f  kube-flannel.yml
clusterrole "flannel" created
clusterrolebinding "flannel" created
serviceaccount "flannel" created
configmap "kube-flannel-cfg" created
daemonset "kube-flannel-ds" created
```

> 这里注意kube-flannel.yml这个文件里的flannel的镜像是0.9.1，quay.io/coreos/flannel:v0.9.1-amd64

如果Node有多个网卡的话，参考[flannel issues 39701](https://github.com/kubernetes/kubernetes/issues/39701)，目前需要在kube-flannel.yml中使用`--iface`参数指定集群主机内网网卡的名称，否则可能会出现dns无法解析。需要将kube-flannel.yml下载到本地，flanneld启动参数加上`--iface=<iface-name>`

```
......
containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.9.1-amd64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=eth1
......

```

使用`kubectl get pod --all-namespaces -o wide`确保所有的Pod都处于Running状态。

```Shell
kubectl get pod --all-namespaces -o wide
```

## 5.master node参与工作负载

使用kubeadm初始化的集群，出于安全考虑Pod不会被调度到Master Node上，也就是说Master Node不参与工作负载。

这里搭建的是测试环境可以使用下面的命令使Master Node参与工作负载：

```Shell
kubectl taint nodes node1 node-role.kubernetes.io/master-
node "node1" untainted
```

## 6.测试DNS

```Shell
kubectl run curl --image=radial/busyboxplus:curl -i --tty
If you don't see a command prompt, try pressing enter.
[ root@curl-2716574283-xr8zd:/ ]$
```

进入后执行nslookup kubernetes.default确认解析正常:

```Shell
nslookup kubernetes.default
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes.default
Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local
```

## 7.向Kubernetes集群添加Node

- 查看token hash值，[官网链接](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-join/)

```shell
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

```Shell
kubeadmin token list
```

- 下面我们将node2这个主机添加到Kubernetes集群中，因为我们同样在node2上的kubelet的启动参数中去掉了必须关闭swap的限制，所以同样需要`--ignore-preflight-errors=Swap`这个参数。 在node2上执行:

```
kubeadm join --token 227d9b.9c1772a7ab45942d 192.168.61.11:6443 --discovery-token-ca-cert-hash sha256:e2aa90853cc97410904adc5d58fbb52f4377ad091a0a807bed0dc69e37107151 \
>   --ignore-preflight-errors=Swap
[preflight] Running pre-flight checks.
        [WARNING Service-Docker]: docker service is not enabled, please run 'systemctl enable docker.service'
        [WARNING Swap]: running with swap on is not supported. Please disable swap
        [WARNING FileExisting-crictl]: crictl not found in system path
[preflight] Starting the kubelet service
[discovery] Trying to connect to API Server "192.168.61.11:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://192.168.61.11:6443"
[discovery] Requesting info from "https://192.168.61.11:6443" again to validate TLS against the pinned public key
[discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "192.168.61.11:6443"
[discovery] Successfully established connection with API Server "192.168.61.11:6443"

This node has joined the cluster:
* Certificate signing request was sent to master and a response
  was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.

```

node2加入集群很是顺利，下面在master节点上执行命令查看集群中的节点：

```Shell
kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
node1     Ready     master    26m       v1.9.0
node2     Ready     <none>    2m        v1.9.0
```

### 如何从集群中移除Node

如果需要从集群中移除node2这个Node执行下面的命令：

在master节点上执行：

```Shell
kubectl drain node2 --delete-local-data --force --ignore-daemonsets
kubectl delete node node2
```

在node2上执行：

```Shell
kubeadm reset
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/
```

## 附录

### 本次安装涉及到的Docker镜像：

```shell
gcr.io/google_containers/kube-proxy-amd64:v1.9.0
gcr.io/google_containers/kube-apiserver-amd64:v1.9.0
gcr.io/google_containers/kube-controller-manager-amd64:v1.9.0
gcr.io/google_containers/kube-scheduler-amd64:v1.9.0
quay.io/coreos/flannel:v0.9.1-amd64
gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.7
gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.7
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.7
gcr.io/google_containers/etcd-amd64:3.1.10
gcr.io/google_containers/pause-amd64:3.0

gcr.io/google_containers/kubernetes-dashboard-amd64:v1.8.1
gcr.io/google_containers/heapster-influxdb-amd64:v1.3.3
gcr.io/google_containers/heapster-grafana-amd64:v4.4.3
gcr.io/google_containers/heapster-amd64:v1.4.2
```

### 配置代理
- <span id="jump">配置全局代理</span>
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
