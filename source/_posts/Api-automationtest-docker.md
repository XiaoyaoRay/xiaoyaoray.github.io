---
title: API自动化测试Docker版使用
date: 2018-11-13 10:00:41
categories:
- Wise2c
tags:
- Wise2c
---

## API自动化测试Docker版使用

### Docker-compose的使用

- 在系统中新建一个目录

  ```
  mkdir -p robot
  cd robot
  ```

  <!--more-->

- 在新建的目录下创建一个`docker-compose.yaml`文件，文件内容如下

  ```
  version: '2'
  services:
    log:
      image: "registry.cn-hangzhou.aliyuncs.com/imagess/wiserobot:log"
      volumes_from:
        - robot
      environment:
        - LOG_DIR_PATH=/data/log
      depends_on:
        - robot
    robot:
      image: "registry.cn-hangzhou.aliyuncs.com/imagess/wiserobot:lib"
      environment:
        - LOG_DIR_PATH=/data/log
        - ROBOT_CONFIG_FILE_PATH=/data/wisecloud.yaml
      volumes:
        - ${PWD}/config:/data
  
  volumes:
    config:
      driver: local
  ```

- 启动docker-compose，启动完成后在当前目录下会自动生成`config`目录

  ```
  docker-compose up -d
  ```



### API自动化测试脚本的启动

#### 目录`config`下文件简介

```
[root@test--002 config]# ls -l
total 8
-rw-r--r-- 1 root root   12 Nov 13 10:03 robot_run_status
-rw-r--r-- 1 root root 2741 Nov 13 09:50 wisecloud.yaml
```

- 文件`robot_run_status`，记录自动化脚本运行的状态

  >  `init`: docker-compose启动后的状态
  >
  > `start`: 启动自动化运行脚本，即如果要运行自动化脚本需要配置为`start`
  >
  > `running`: 自动化脚本执行过程中
  >
  >  `end`: 自动化脚本执行完成，在该状态是会自动提交运行后的日志到码云上
  >
  > `git log end`: 提交日志到码云操作完成

- 文件`wisecloud.yaml`，自动化脚本运行所有的配置文件


#### 启动自动化脚本

```
echo "start" > config/robot_run_status
docker-compose logs -f
```

#### 查看运行后日志

#####  测试环境

- 报告链接

  <https://wisecloud.gitee.io/automationlog/test/report.html>

- 日志链接

  <https://wisecloud.gitee.io/automationlog/test/log.html>

##### 开发环境

- 报告链接

  <https://wisecloud.gitee.io/automationlog/dev/report.html>

- 日志链接

  <https://wisecloud.gitee.io/automationlog/dev/log.html>

##### 金科

- 报告链接

  <https://wisecloud.gitee.io/automationlog/jinke/report.html>

- 日志链接

  <https://wisecloud.gitee.io/automationlog/jinke/log.html>



### 使用Makefile重新执行自动化脚本

#### `Makefile` 文件

存放位置，与`docker-compose.yaml` 文件同级目录

```
restart:
	docker pull registry.cn-hangzhou.aliyuncs.com/imagess/wiserobot:lib
	docker pull registry.cn-hangzhou.aliyuncs.com/imagess/wiserobot:log
	docker-compose down
	rm -rf config
	docker-compose up -d
	echo "start" > config/robot_run_status
	docker-compose logs -f
```

#### 运行命令

```
make restart
```

