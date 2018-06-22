---
title: 使用Docker安装Testlink
copyright: true
date: 2018-06-06 10:18:55
tags:
- Testlink
---

### Docker-compose安装

#### docker-compose.yaml文件

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

```shell
docker-compose up -d
docker-compose logs -f
```

###  docker-compose重新安装

```shell
docker-compose down
docker volume list

# 删除testlink的两个数据卷
docker volume rm <testlink_mariadb_data testlink_testlink_data>
```

#### 备份数据

```shell
[root@test--0005 volumes]# pwd
/var/lib/docker/volumes

# 需要备份的两个数据
[root@test--0005 volumes]# ls | grep testlink
testlink_mariadb_data
testlink_testlink_data
[root@test--0005 volumes]#
```



### 直接用docker安装

#### 创建testlink的docker网络

```shell
docker network create testlink-tier
```

#### 安装数据库

```shell
docker run -d --name mariadb \
--net testlink-tier \
-e ALLOW_EMPTY_PASSWORD=yes \
-e MARIADB_USER=bn_testlink \
-e MARIADB_DATABASE=bitnami_testlink \
--volume ~/testlink/mariadb_data:/bitnami \
bitnami/mariadb:latest
```

#### 安装Testlink

```shell
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