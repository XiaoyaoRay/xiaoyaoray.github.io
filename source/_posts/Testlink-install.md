---
title: 使用Docker安装Testlink
copyright: true
date: 2018-06-06 10:18:55
tags:
- Testlink
---

### Docker-compose安装

#### docker-compose.yaml文件
<!--more-->
```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=bn_testlink
      - MARIADB_DATABASE=bitnami_testlink
    volumes:
      - 'mariadb_data:/bitnami'
  testlink:
    image: 'bitnami/testlink:latest'
    ports:
      - '10080:80'
      - '10443:443'
    volumes:
      - 'testlink_data:/bitnami'
    depends_on:
      - mariadb
    environment:
      - MARIADB_HOST=mariadb
      - MARIADB_PORT_NUMBER=3306
      - TESTLINK_DATABASE_USER=bn_testlink
      - TESTLINK_DATABASE_NAME=bitnami_testlink
      - ALLOW_EMPTY_PASSWORD=yes
      - TESTLINK_USERNAME=admin
      - TESTLINK_PASSWORD=wise2c2018
      - TESTLINK_EMAIL=admin@wise2c.com
      - TESTLINK_LANGUAGE=zh_CN

volumes:
  mariadb_data:
    driver: local
  testlink_data:
    driver: local
```

#### 命令使用

```
docker-compose up -d
docker-compose logs -f
```

###  docker-compose重新安装

```
docker-compose down
docker volume list

# 删除testlink的两个数据卷
docker volume rm <testlink_mariadb_data testlink_testlink_data>
```

### 备份数据

```
[root@test--0005 volumes]# pwd
/var/lib/docker/volumes

# 需要备份的数据
[root@test--0005 volumes]# ls | grep testlink
testlink_mariadb_data

# 打包
[root@test--0005 volumes]#cd /var/lib/docker/volumes/testlink_mariadb_data/_data
[root@test--0005 _data]#tar -czf mariadb_data.tar.gz .

```

#### 恢复数据

```
# 新建目录存放数据
mkdir mariadb_data

# 把备份的数据解压到上述文件中
tar -xvfz mariadb_data.tar.gz -C mariadb_data

# 修改docker-compose.yaml中数据卷
.....
....
 mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=bn_testlink
      - MARIADB_DATABASE=bitnami_testlink
    volumes:
      - '/root/mariadb_data:/bitnami'	# 修改为创建的实际路径
  testlink:
    image: 'bitnami/testlink:latest'
    ports:
      - '10080:80'
      - '10443:443'
    volumes:
      - 'testlink_data:/bitnami'
    depends_on:
      - mariadb
....
....

# 启动docker-compose
docker-compose up -d
```

#### 每天定时备份脚本

```
#!/bin/sh

# 运行命令,如果失败会尝试3次
run_cmd(){
    for i in `seq 3`  
    do  
        $1
        if [ $? -eq 0 ];then
            break
        fi  
    done 
}

gitdir="/home/testlink"

# 从git上拉取自动化测试代码
run_cmd "git clone https://<码云账号>:<账号密码>@gitee.com/wisecloud/wisecloud-test.git $gitdir"

# 切换到testlink分支 日志输出文件以时间来命名
cd $gitdir
git checkout testlink
outputdir=`date +%Y%m%d%H%M%S`

# testlink 备份数据打包
echo "mariadb data backup---------------------------"
cd /var/lib/docker/volumes/testlink_mariadb_data/_data
tar -czf mariadb_data_${outputdir}.tar.gz .
mv mariadb_data_${outputdir}.tar.gz $gitdir

# 保留8个以内的日志文件
cd $gitdir
dirs_mariadb=`ls -l ./ | grep "mariadb_data_" | awk '{print $9}'| sort -r`
i=0
for name in $dirs_mariadb
do
    let i=i+1
    if [ $i -ge 8 ];then
        rm -rf $name
	echo "delete $name -----------"
    fi
done

# 提交到码云上
git config --global user.email "<邮件地址>"
git config --global user.name "TestGroup"
git add --all
git commit -m $outputdir
run_cmd "git push origin testlink"

rm -rf $gitdir
```



### 直接用docker安装

#### 创建testlink的docker网络

```
docker network create testlink-tier
```

#### 安装数据库

```
docker run -d --name mariadb \
--net testlink-tier \
-e ALLOW_EMPTY_PASSWORD=yes \
-e MARIADB_USER=bn_testlink \
-e MARIADB_DATABASE=bitnami_testlink \
--volume ~/testlink/mariadb_data:/bitnami \
bitnami/mariadb:latest
```

#### 安装Testlink

```
docker run -d -p 18000:80 -p 18443:443 --name testlink \
-e ALLOW_EMPTY_PASSWORD=yes \
-e MARIADB_USER=bn_testlink \
-e MARIADB_DATABASE=bitnami_testlink \
--net testlink-tier \
--volume ~/testlink/testlink_data:/bitnami \
bitnami/testlink:latest
```

**PS**:网页登录的默认密码：`user/bitnami`

### 参考

https://github.com/bitnami/bitnami-docker-testlink
