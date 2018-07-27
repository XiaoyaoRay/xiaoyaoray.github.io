---
title: Kubernetes安装之Kubespray
copyright: true
date: 2018-06-22 13:58:01
tags:
- Kubernetes
---

### 基于kubespray安装k8s集群

- **使用分支是v2.4.0**

  https://github.com/kubernetes-incubator/kubespray/tree/v2.4.0

- **组件安装的版本**

  kubernetes v1.9.2

  etcd v3.2.4

  calico v2.5.0

  docker v1.13
<!--more-->
### 安装步骤

- 主机配置
  - 内存：最少大于1.5G，否则ansible安装时会报错

- 登入三台机器关闭防火墙

  ```shell
  # 每台主机关闭防火墙
  systemctl stop firewalld
  systemctl disable firewalld
  
  # 每台主机关闭swap
  swapoff -a
  ```

- 登入ansible host

  ```shell
  # 安装yum的epel源
  yum install -y epel-release
  
  # 安装需求的软件
  yum install -y python-pip python-netaddr ansible git
  pip install --upgrade Jinja2
  
  # 配置ssh免密
  ssh-keygen
  ssh-copy-id root@172.31.19.29
  ssh-copy-id root@172.31.19.30
  ssh-copy-id root@172.31.19.31
  
  # 拉取kubespray 并切换分支
  git clone https://github.com/kubernetes-incubator/kubespray.git
  cd kubespray
  git checkout v2.4.0 -b myv2.4.0
  ```

- 修改inventory

  ```shell
  # 复制主机配置
  cp inventory/inventory.example inventory/inventory.cfg
  vim inventory/inventory.cfg
  [all]
  node1 ansible_ssh_host=172.31.19.29 ip=172.31.19.29
  node2 ansible_ssh_host=172.31.19.30 ip=172.31.19.30
  node3 ansible_ssh_host=172.31.19.31 ip=172.31.19.31
  
  [kube-master]
  node1
  node2
  node3
  
  [etcd]
  node1
  node2
  node3
  
  [kube-node]
  node1
  node2
  node3
  
  [k8s-cluster:children]
  kube-node
  kube-master
  ```

- 备份并修改安装的配置

  ```shell
  # 备份
  cp inventory/group_vars/all.yml inventory/group_vars/all.yml.bak
  cp inventory/group_vars/k8s-cluster.yml inventory/group_vars/k8s-cluster.yml.bak
  
  # 修改bootstrap版本
  vim inventory/group_vars/all.yml
  bootstrap_os: centos
  
  # 关闭Dashboard和修改kube_api_pwd密码
  vim inventory/group_vars/k8s-cluster.yml
  dashboard_enabled: false
  kube_api_pwd: “hello-world8888”
  ```

- 修改下载的包，将国外下载慢的包，改成用阿里的

  ```shell
  # 修改roles/download/defaults/main.yml中的镜像
  sed -i 's#^etcd_image_repo:.*#etcd_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/etcd"#g' roles/download/defaults/main.yml
  sed -i 's#^calicoctl_image_repo:.*#calicoctl_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/ctl"#g' roles/download/defaults/main.yml
  sed -i 's#^calico_node_image_repo:.*#calico_node_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/node"#g' roles/download/defaults/main.yml
  sed -i 's#^calico_cni_image_repo:.*#calico_cni_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/cni"#g' roles/download/defaults/main.yml
  sed -i 's#^calico_policy_image_repo:.*#calico_policy_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/kube-policy-controller"#g' roles/download/defaults/main.yml
  sed -i 's#^calico_rr_image_repo:.*#calico_rr_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/routereflector"#g' roles/download/defaults/main.yml
  sed -i 's#^hyperkube_image_repo:.*#hyperkube_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/hyper-kube"#g' roles/download/defaults/main.yml
  sed -i 's#^pod_infra_image_repo:.*#pod_infra_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/pause-amd64"#g' roles/download/defaults/main.yml
  sed -i 's#^nginx_image_repo:.*#nginx_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/nginx"#g' roles/download/defaults/main.yml
  sed -i 's#^kubedns_image_repo:.*#kubedns_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/k8s-dns-kube-dns-amd64"#g' roles/download/defaults/main.yml
  sed -i 's#^dnsmasq_nanny_image_repo:.*#dnsmasq_nanny_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/k8s-dns-dnsmasq-nanny-amd64"#g' roles/download/defaults/main.yml
  sed -i 's#^dnsmasq_sidecar_image_repo:.*#dnsmasq_sidecar_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/k8s-dns-sidecar-amd64"#g' roles/download/defaults/main.yml
  sed -i 's#^kubednsautoscaler_image_repo:.*#kubednsautoscaler_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/cluster-proportional-autoscaler-amd64"#g' roles/download/defaults/main.yml
  
  # 修改roles/kubernetes-apps/ansible/defaults/main.yml文件的镜像源
  sed -i 's#^kubedns_image_repo:.*#kubedns_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/k8s-dns-kube-dns-amd64"#g' roles/kubernetes-apps/ansible/defaults/main.yml
  sed -i 's#^dnsmasq_nanny_image_repo:.*#dnsmasq_nanny_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/k8s-dns-dnsmasq-nanny-amd64"#g' roles/kubernetes-apps/ansible/defaults/main.yml
  sed -i 's#^dnsmasq_sidecar_image_repo:.*#dnsmasq_sidecar_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/k8s-dns-sidecar-amd64"#g' roles/kubernetes-apps/ansible/defaults/main.yml
  sed -i 's#^kubednsautoscaler_image_repo:.*#kubednsautoscaler_image_repo: "registry.cn-hangzhou.aliyuncs.com/linkcloud/cluster-proportional-autoscaler-amd64"#g' roles/kubernetes-apps/ansible/defaults/main.yml
  
  ```

- 部署

  ```shell
  ansible-playbook -b -i inventory/inventory.cfg cluster.yml --flush-cache
  ```
