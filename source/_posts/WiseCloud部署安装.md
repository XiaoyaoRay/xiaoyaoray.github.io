---
title: WiseCloud部署安装
copyright: true
date: 2018-05-23 14:26:35
tags:
- Wise2c
---

### WiseCloud部署安装

#### 部署UI搭建

- 新建目录` deploy-ui`，在该目录下新建文件`docker-compose.yml`，内容如下：

  ```yaml
  version: '2'
  services:
    deploy:
      image: wise2c/pagoda:v0.4.1
      restart: always
      entrypoint: sh
      command:
      - -c
      - "/root/pagoda -logtostderr -v 4 -w /workspace"
      ports:
      - 88:80
      - 8088:8080
      volumes:
      - /root/.ssh:/root/.ssh
      - $PWD/deploy:/deploy
      volumes_from:
      - wise2c-playbook
    ui:
      image: registry.cn-hangzhou.aliyuncs.com/wise2c-dev/deploy-ui:v0.4
      restart: always
      network_mode: "service:deploy"
    wise2c-playbook:
      image: registry.cn-hangzhou.aliyuncs.com/wise2c-dev/playbook:v1.4.10-rc1
      volumes:
      - wise2c-playbook:/workspace
    yum-repo:
      image: wise2c/yum-repo:7.4
      ports:
      - 2009:2009
      restart: always
  volumes:
    wise2c-playbook:
      external: false
  ```

- 在目录`deploy-ui`目录下运行命令：

  ```shell
  # 清理已存在volume
  [root@test--0005 deploy-ui]# docker volume list | grep wise2c-playbook
  local               deployui_wise2c-playbook
  [root@test--0005 deploy-ui]# docker volume rm deployui_wise2c-playbook
  
  # 启动部署UI的docker-compose
  [root@test--0005 deploy-ui]# docker-compose up -d
  Creating network "deployui_default" with the default driver
  Creating deployui_wise2c-playbook_1 ...
  Creating deployui_yum-repo_1 ...
  Creating deployui_yum-repo_1
  Creating deployui_wise2c-playbook_1 ... done
  Creating deployui_deploy_1 ...
  Creating deployui_deploy_1 ... done
  Creating deployui_ui_1 ...
  Creating deployui_ui_1 ... done
  
  # 查看是否启动成功
  [root@test--0005 deploy-ui]# docker-compose ps
             Name                         Command               State                      Ports
  -----------------------------------------------------------------------------------------------------------------
  deployui_deploy_1            sh -c /root/pagoda -logtos ...   Up       0.0.0.0:88->80/tcp, 0.0.0.0:8088->8080/tcp
  deployui_ui_1                /root/entrypoint.sh              Up
  deployui_wise2c-playbook_1   sh                               Exit 0
  deployui_yum-repo_1          nginx -g daemon off;             Up       0.0.0.0:2009->2009/tcp, 80/tcp
  [root@test--0005 deploy-ui]#
  ```

- 部署`playbook`存放目录

  ```shell
  # 查看playbook的映射到宿主机的目录名称
  [root@test--0005 deploy-ui]# docker volume list | grep wise2c-playbook
  local               deployui_wise2c-playbook
  [root@test--0005 deploy-ui]#
  
  # 宿主机上playbook的路径
  [root@test--0005 deploy-ui]# ls /var/lib/docker/volumes/deployui_wise2c-playbook/_data/
  callback_plugins  components_order.conf  consul-playbook  docker-playbook  etcd-playbook  kubernetes-playbook  loadbalancer-playbook  mysql-playbook  rabbitmq-playbook  redis-playbook  registry-playbook  wisecloud-playbook
  [root@test--0005 deploy-ui]#
  ```

#### 部署UI安装各个组件

- 访问地址`http://<docker-compose宿主机IP>:88`，端口可以在docker-compose.yml文件中配置

  ![image-20180523111531540](https://ws3.sinaimg.cn/large/006tNc79gy1frl3x76gf2j30po08o74y.jpg)

- 页面主机配置

  ![image-20180523111601694](https://ws2.sinaimg.cn/large/006tNc79gy1frl3x4050vj31kw0srn17.jpg)

- 页面服务组件配置

  ![image-20180523111850381](https://ws2.sinaimg.cn/large/006tNc79gy1frl3wzqrarj31ai0nnq6m.jpg)

- 勾选需要安装组件后，在服务组件点击`开始安装`按钮镜像部署

  ![image-20180523112119274](https://ws4.sinaimg.cn/large/006tNc79gy1frl3wvfv9mj30kj0ayq3o.jpg)

- 安装时用命令查看日志

  ```shell
  #查看部署日志
  [root@test--0005 deploy-ui]# docker-compose logs -f deploy
  Attaching to deployui_ui_1, deployui_deploy_1, deployui_wise2c-playbook_1, deployui_yum-repo_1
  deploy_1           | [GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
  deploy_1           |  - using env:	export GIN_MODE=release
  deploy_1           |  - using code:	gin.SetMode(gin.ReleaseMode)
  deploy_1           |
  ui_1               | Activating privacy features... done.
  deploy_1           | [GIN-debug] GET    /favicon.ico              --> github.com/wise2c-devops/pagoda/vendor/github.com/gin-gonic/gin.(*RouterGroup).StaticFile.func1 (4 handlers)
  ui_1               | http://0.0.0.0
  deploy_1           | [GIN-debug] HEAD   /favicon.ico              --> github.com/wise2c-devops/pagoda/vendor/github.com/gin-gonic/gin.(*RouterGroup).StaticFile.func1 (4 handlers)
  ui_1               | http://0.0.0.0/v1
  deploy_1           | [GIN-debug] GET    /v1/components            --> main.components (4 handlers)
  ui_1               | 171.221.145.40 - - [23/May/2018:03:11:57 +0000] "GET /v1/clusters HTTP/1.1" 200 113
  deploy_1           | [GIN-debug] GET    /v1/components/:component_name/versions --> main.versions (4 handlers)
  ui_1               | 171.221.145.40 - - [23/May/2018:03:11:59 +0000] "GET /v1/clusters/ee1477ca-e751-45f6-96f2-44708ded91f8/hosts HTTP/1.1" 200 305
  deploy_1           | [GIN-debug] GET    /v1/components/:component_name/properties/:version --> main.properties (4 handlers)
  deploy_1           | [GIN-debug] GET    /v1/clusters              --> main.retrieveClusters (4 handlers)
  ui_1               | 171.221.145.40 - - [23/May/2018:03:11:59 +0000] "GET /v1/clusters/ee1477ca-e751-45f6-96f2-44708ded91f8 HTTP/1.1" 200 981
  deploy_1           | [GIN-debug] POST   /v1/clusters              --> main.createCluster (4 handlers)
  deploy_1           | [GIN-debug] DELETE /v1/clusters/:cluster_id  --> main.deleteCluster (4 handlers)
  deploy_1           | [GIN-debug] PUT    /v1/clusters/:cluster_id  --> main.updateCluster (4 handlers)
  ui_1               | 171.221.145.40 - - [23/May/2018:03:17:49 +0000] "GET /v1/clusters/ee1477ca-e751-45f6-96f2-44708ded91f8/components HTTP/1.1" 200 926
  ```

- 直接在docker容器中运行playbook可以看到更具体的信息，在容器内找到playbook的`install.ansible`文件，运行命令`ansible-playbook install.ansible`

  ```shell
  # 查看部署UI的docker容器
  [root@test--0005 deploy-ui]# docker ps | grep deploy
  9f3927ba11b5        registry.cn-hangzhou.aliyuncs.com/wise2c-dev/deploy-ui:v0.4                                                                  "/root/entrypoint.sh"    31 minutes ago      Up 31 minutes                                                                                                                                                deployui_ui_1
  6064cfd73cc5        wise2c/pagoda:v0.4.1                                                                                                         "sh -c '/root/pagoda "   31 minutes ago      Up 31 minutes           0.0.0.0:88->80/tcp, 0.0.0.0:8088->8080/tcp                                                                                           deployui_deploy_1
  1c153575e397        wise2c/yum-repo:7.4                                                                                                          "nginx -g 'daemon off"   31 minutes ago      Up 31 minutes           80/tcp, 0.0.0.0:2009->2009/tcp                                                                                                       deployui_yum-repo_1
  
  # 进入到docker容器中
  [root@test--0005 deploy-ui]# docker exec -it deployui_deploy_1 sh
  /deploy # cd /workspace/
  /workspace # ls
  callback_plugins       consul-playbook        etcd-playbook          loadbalancer-playbook  rabbitmq-playbook      registry-playbook
  components_order.conf  docker-playbook        kubernetes-playbook    mysql-playbook         redis-playbook         wisecloud-playbook
  
  # 进入到某个playbook目录中
  /workspace # cd kubernetes-playbook/
  /workspace/kubernetes-playbook # ls
  v1.8.6
  /workspace/kubernetes-playbook # cd v1.8.6/
  /workspace/kubernetes-playbook/v1.8.6 #
  /workspace/kubernetes-playbook/v1.8.6 # ls
  ansible.cfg      group_vars       inherent.yaml    install.ansible  node.ansible     reset.ansible    template
  file             hosts            init.sh          master.ansible   properties.json  scripts          yat
  /workspace/kubernetes-playbook/v1.8.6 #
  
  # 运行playbook
  /workspace/kubernetes-playbook/v1.8.6 # ansible-playbook install.ansible
  ```

  

#### 部署WiseCloud

- 新建`img-list.txt`,`pull-img.sh`,`push-img.sh`

  - `img-list.txt`内容

    ```tex
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/conf_agent:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/wise-logger:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/metrics:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/build-biz:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/ingress-controller:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/confcenter:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/wisecloud-orchestration:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/tosca-api:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/ingress-agent:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/ingress-gateway:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/ingress-gateway:stable
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/tosca-translator:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/wisebuild-fronted:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/fluentd:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/build-worker:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/prometheus-rest:latest
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/wisecloud-autoscaling:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/resource-manager:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/host-agent:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/monitor-api:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/catalog:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/user-manual:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/service-register:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/conf_init:stable
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/build-schedule:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/apigateway:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/metadata:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/backup-restore:v1.4.10
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/nfs-provisioner:v0.8.1
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/kibana:cloud-6.1.1
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/curator:latest
    registry.cn-hangzhou.aliyuncs.com/wise2c-test/cdflow:v1.4.10
    docker.io/wisecity/kubernetes-exporter:latest
    docker.io/wisecity/artifactory-oss:5.2.0
    docker.io/wisecity/docker-metrics-exporter:latest
    prom/node-exporter:latest
    prom/alertmanager:v0.7.1
    prom/prometheus:v1.7.1
    wisecity/prometheus-adaptor
    docker.io/influxdb:1.3.5
    google/cadvisor
    docker.elastic.co/elasticsearch/elasticsearch:6.1.1
    antillion/slanger:latest
    docker.io/progrium/consul:latest
    docker.io/redis:4.0.8-alpine
    docker.io/rabbitmq:3.7
    kuberstack/kubernetes-rabbitmq-autocluster:latest
    docker.io/library/busybox:latest
    docker.io/centos:7
    docker.io/nginx:latest
    docker.io/byrnedo/alpine-curl:latest
    ```

  - `pull-img.sh`内容

    ```shell
    #!/bin/bash
    
    function pull_img(){
        imgNumPerTar=3
        i=1
        pkg_idx=1
        images=""
    
        while read line
        do
      if [[ $(echo ${line} | wc -L) == 0 ]]; then
        echo "+++++++++++ Note: found space line, skip it! +++++++++++"
        continue
        echo "hahah"
      fi
            new_label=${registry_url}"/"${registry_project}"/"${line##*/}
            docker pull $new_label
            if [ $? -ne 0 ]; then
                echo "[NOTE]: pull image $new_label failed!"
            fi
            i=$(($i+1))
        done < ./img-list.txt
    }
    
    function docker_login(){
        docker login -u=$registry_user_name -p=$registry_password $registry_url
        if [ $? -ne 0 ]; then
            echo "[NOTE]: docker login $registry_url failed!"
            return 2
        fi
    }
    
    registry_url="registry.cn-hangzhou.aliyuncs.com"
    registry_project="wise2c-test"
    registry_user_name="wise2cdev"
    registry_password="Wise2c2017"
    registry_email="wise2c@aliyun.com"
    
    echo "args: " $registry_url $registry_project $registry_user_name $registry_password $registry_email
    
    docker_login
    pull_img
    [root@test--
    ```

  - `push-img.sh`内容

    ```shell
    #!/bin/bash
    
    
    function push_img(){
        all_images=$(while read line;do echo $line;done <./img-list.txt)
        images=$(echo $all_images | tr " " "\n")
        for img in $images
        do
                echo $img
    	    old_label=${source_registry_url}"/"${source_registry_project}"/"${img##*/}
                new_label=${registry_url}"/"${registry_project}"/"${img##*/}
                echo "old_label:" $old_label
                echo "new_label:" $new_label
                docker tag $old_label $new_label
                docker push $new_label
        done
    }
    
    function docker_login(){
        docker login -u=$registry_user_name -p=$registry_password $registry_url
        if [ $? -ne 0 ]; then
            echo "[NOTE]: docker login $registry_url failed!"
            return 2
        fi
    }
    
    source_registry_url="registry.cn-hangzhou.aliyuncs.com"
    source_registry_project="wise2c-test"
    
    
    registry_url="192.168.1.251"
    registry_project="wise2c"
    registry_user_name="admin"
    registry_password="Harbor12345"
    registry_email="wise2c@aliyun.com"
    
    
    echo "args: " $registry_url $registry_project $registry_user_name $registry_password
    
    docker_login
    push_img
    ```

    

- 把需要的镜像push到对应的registry(Harbor)

  ```shell
  # 需要的文件列表
  [root@test--0004 v1.4.10]# ls
  img-list.txt  pull-img.sh  push-img.sh
  [root@test--0004 v1.4.10]#
  
  # 从wise2c-test空间pull镜像到本地
  [root@test--0004 v1.4.10]# ./pull-img.sh
  args:  registry.cn-hangzhou.aliyuncs.com wise2c-test wise2cdev Wise2c2017 wise2c@aliyun.com
  Login Succeeded
  Trying to pull repository registry.cn-hangzhou.aliyuncs.com/wise2c-test/conf_agent ...
  sha256:667ab0ce12ce53b7cd7a527c0620ce05e197d924e658dc37d4c9ec37fe4d6cda: Pulling from registry.cn-hangzhou.aliyuncs.com/wise2c-test/conf_agent
  Digest: sha256:667ab0ce12ce53b7cd7a527c0620ce05e197d924e658dc37d4c9ec37fe4d6cda
  ........
  Status: Image is up to date for registry.cn-hangzhou.aliyuncs.com/wise2c-test/alpine-curl:latest
  +++++++++++ Note: found space line, skip it! +++++++++++
  [root@test--0004 v1.4.10]#
  
  # 把镜像push的内置镜像仓库
  [root@test--0004 v1.4.10]# ./push-img.sh
  args:  192.168.1.251 wise2c admin Harbor12345
  Login Succeeded
  registry.cn-hangzhou.aliyuncs.com/wise2c-test/conf_agent:v1.4.10
  old_label: registry.cn-hangzhou.aliyuncs.com/wise2c-test/conf_agent:v1.4.10
  new_label: 192.168.1.251/wise2c/conf_agent:v1.4.10
  The push refers to a repository [192.168.1.251/wise2c/conf_agent]
  25620950a61d: Layer already exists
  cd7100a72410: Layer already exists
  v1.4.10: digest: sha256:667ab0ce12ce53b7cd7a527c0620ce05e197d924e658dc37d4c9ec37fe4d6cda size: 740
  registry.cn-hangzhou.aliyuncs.com/wise2c-test/wise-logger:v1.4.10
  old_label: registry.cn-hangzhou.aliyuncs.com/wise2c-test/wise-logger:v1.4.10
  new_label: 192.168.1.251/wise2c/wise-logger:v1.4.10
  ......
  1ee6de2057d0: Layer already exists
  cd7100a72410: Layer already exists
  latest: digest: sha256:00c64bda2814f95a13ad6a46d6de487dc88f424f2b1f6c62c6dfce36701fe516 size: 738
  [root@test--0004 v1.4.10]#
  ```

- 从码云上拉取部署需要的文件

  ```shell
  [root@test--0004 ~]# git clone https://gitee.com/wisecloud/wisecloud-k8s-yaml.git
  Cloning into 'wisecloud-k8s-yaml'...
  Username for 'https://gitee.com': leicj@wise2c.com
  Password for 'https://leicj@wise2c.com@gitee.com':
  remote: Counting objects: 3085, done.
  remote: Compressing objects: 100% (2613/2613), done.
  remote: Total 3085 (delta 2098), reused 668 (delta 439)
  Receiving objects: 100% (3085/3085), 443.03 KiB | 0 bytes/s, done.
  Resolving deltas: 100% (2098/2098), done.
  [root@test--0004 ~]#
  
  # 切换对应的分支
  [root@test--0004 wisecloud-k8s-yaml]# git checkout v1.4.10	
  Branch v1.4.10 set up to track remote branch v1.4.10 from origin.
  Switched to a new branch 'v1.4.10'
  [root@test--0004 wisecloud-k8s-yaml]#
  ```

- 为主机打上标签

  ```shell
  # 进入到wisecloud-k8s-yaml目录
  [root@test--0004 wisecloud-k8s-yaml]# pwd
  /root/liaoxb/v1.4.10/wisecloud-k8s-yaml
  [root@test--0004 wisecloud-k8s-yaml]#
  
  # 运行打标签命令
  [root@test--0004 wisecloud-k8s-yaml]#./preDeploy.py --file label.json
  ```

  `label.json`文件内容

  ```shell
  {"roles":[
      {"name":"controller", "hosts":["test--0001","test--0002","test--0003"]},
      {"name":"ingress-gateway", "hosts":["test--0004","test--0006"]},
      {"name":"prometheus",      "hosts":["test--0002"]},
      {"name":"artifactory",     "hosts":["test--0003"]},
      {"name":"kibana",          "hosts":["test--0001"]},
      {"name":"buildflow",       "hosts":["test--0001"]}
  ]}
  ```

- 执行部署脚本

  ```shell
  # 进入到wisecloud-k8s-yaml目录
  [root@test--0004 wisecloud-k8s-yaml]# pwd
  /root/liaoxb/v1.4.10/wisecloud-k8s-yaml
  [root@test--0004 wisecloud-k8s-yaml]#
  
  # 运行部署脚本
  [root@test--0004 wisecloud-k8s-yaml]#./install.sh
  ```

  `install.sh`文件内容

  ```shell
  # 注意k8s_wisecloud_deploy2.sh 这个文件名称可能有变动
  ./k8s_wisecloud_deploy2.sh \
        --db_url=192.168.1.28 \
        --db_port=3307 \
        --db_user=wise2c \
        --db_pwd=wise2c \
        --etcd=http://192.168.1.28:2379,http://192.168.1.227:2379,http://192.168.1.120:2379 \
        --k8s_endpoint=192.168.1.227:6444 \
        --k8s_hosts=test-002,test-003,test-004 \
        --wisecloud_api_url=192.168.1.227 \
        --wisecloud_api_port=8080 \
        --artifactory_url=192.168.1.227 \
        --artifactory_port=32778 \
        --custom_registry_url=192.168.1.251 \
        --custom_registry_ns=wise2c \
        --custom_registry_user=admin \
        --custom_registry_pwd=Harbor12345 \
        --custom_registry_email=wise2c@aliyun.com \
        --es_url=192.168.1.227 \
        --es_port=32768 \
        --es_cluster_port=32769 \
        --consul_url=192.168.1.227 \
        --consul_port=32772 \
        --redis_url=192.168.1.227 \
        --redis_port=32770 \
        --pusher_url=192.168.1.227 \
        --pusher_api_port=32774 \
        --pusher_ws_port=32773 \
        --mq_url=192.168.1.227 \
        --mq_port=32775 \
        --mq_admin_port=32776 \
        --mode=deploy \
        --custom_install_interval=1
  ```

- 查看部署结果（如果都是running状态，表示部署成功）

  ```shell
  # kcc 命令等效于：kubectl -n wisecloud-controller
  [root@test--0004 ~]# kcc get pods
  NAME                                READY     STATUS    RESTARTS   AGE
  artifactory-7fc884dbcc-xhfjw        1/1       Running   0          3h
  backup-restore-5c6c797cdc-rt6j6     1/1       Running   0          3h
  buildflow-cd8f4b895-92x2t           1/1       Running   0          2h
  config-center-664496955-677hv       1/1       Running   0          3h
  config-center-664496955-h2x7t       1/1       Running   0          3h
  config-center-664496955-tv9td       1/1       Running   0          3h
  elasticsearch-7cc6d9d568-jxr2w      1/1       Running   0          3h
  elasticsearch-7cc6d9d568-k4nqw      1/1       Running   0          3h
  elasticsearch-7cc6d9d568-mhgc5      1/1       Running   0          3h
  kibana-7f964f9877-c9hqc             2/2       Running   0          3h
  metrics-7bd486564f-mcd5m            1/1       Running   0          3h
  metrics-7bd486564f-w4zrj            1/1       Running   0          3h
  metrics-7bd486564f-x7l2z            1/1       Running   0          3h
  monitor-api-c59c57c57-hrghr         1/1       Running   0          3h
  monitor-api-c59c57c57-zwm75         1/1       Running   0          3h
  monitor-api-c59c57c57-zz4rh         1/1       Running   0          3h
  orchestration-6c66f646b4-c8zfz      6/6       Running   0          3h
  orchestration-6c66f646b4-pbb5m      6/6       Running   0          3h
  orchestration-6c66f646b4-r7njz      6/6       Running   0          3h
  prometheus-65d97c9d54-fl4js         7/7       Running   0          3h
  pusher-8566dfff6-4wjkf              2/2       Running   0          3h
  pusher-8566dfff6-jj27m              2/2       Running   0          3h
  pusher-8566dfff6-w4mpg              2/2       Running   0          3h
  resource-manager-58c64446f5-7k2qm   1/1       Running   0          3h
  resource-manager-58c64446f5-pbm4p   1/1       Running   0          3h
  resource-manager-58c64446f5-wclhl   1/1       Running   0          3h
  ui-bc465d74-5xmlm                   3/3       Running   1          3h
  ui-bc465d74-n24h4                   3/3       Running   1          3h
  ui-bc465d74-rt6x6                   3/3       Running   1          3h
  worker-7fm68                        1/1       Running   0          3h
  worker-9ljrb                        1/1       Running   0          3h
  worker-p7gtc                        1/1       Running   0          3h
  workflow-f4c79cdd6-5bwr2            2/2       Running   0          3h
  workflow-f4c79cdd6-9p2p7            2/2       Running   0          3h
  workflow-f4c79cdd6-bhb24            2/2       Running   0          3h
  [root@test--0004 ~]#
  ```

  

### 问题总结

#### 部署中重置相关问题

- Q：在部署ui上重置docker和Kubernetes后，在主机上还能运行docker 和 kubectl命令 ?

  A：重置只是把相关的配置清理掉，不会卸载 。并且重置户安装新的docker版本或者Kubernetes 能安装成功。如果有安装有问题时，可以尝试运行命令卸载`yum remove <package>`

#### 安装部署包

- Q：怎么查看已有的包？

  A：浏览器中访问`http://<部署机器IP>:2009`，出现以下界面

  ![image-20180523140329943](https://ws3.sinaimg.cn/large/006tNc79gy1frl8l5yzv9j311k0x8jvx.jpg)

#### kcc运行pod启动异常

- Q：用命令`kcc get pods`查看发现许多pod状态为`Init:0/1`，并且查看pusher的pod有图片中错误信息

  ![image-20180523140805028](https://ws1.sinaimg.cn/large/006tNc79gy1frl8pz7ed1j30gt0imtdd.jpg)

  ![image-20180523140741207](https://ws3.sinaimg.cn/large/006tNc79gy1frl8pi9ww5j313b0hhdko.jpg)

A：在haproxy所在的主机修改`haproxy.cfg`配置文件，对应的`Redis`,`consul`,`RabbitMQ`字段下server的IP地址改为公共组件所在的地址，然后重启haproxy的docker

```shell
[root@test--0002 loadbalancer]# pwd
/var/lib/wise2c/loadbalancer
[root@test--0002 loadbalancer]#

# 修改haproxy.cfg文件，`Redis`,`consul`,`RabbitMQ`字段下有server IP的均有修改
[root@test--0002 loadbalancer]# vi haproxy.cfg
```

