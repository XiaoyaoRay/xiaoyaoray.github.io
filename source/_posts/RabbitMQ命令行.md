---
title: RabbitMQ命令行
copyright: true
tags:
- RabbitMQ
---

## Docker运行RabbitMQ

```shell
docker run -d --name rabbitmq -v $PWD/:/var/lib/rabbitmq --hostname my-rabbit -p 5672:5672 -p 4369:4369 -p 8181:15672 rabbitmq:3-management
```

guest:guest

- docker exec -ti <rabbitmq> /bin/bash
- rabbitmqctl trace_on
- rabbitmq-plugins enable rabbitmq_tracing
- exit

## RabbitMQ命令

[wise2c链接](https://gitee.com/wisecloud/xing/blob/master/doc/starting_cn.md)

### 查看所有队列

```shell
rabbitmqctrl list_queues
```

### 清除命令

```shell
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
```

### 添加用户，给权限

```shell
rabbitmqctl list_users
rabbitmqctl add_user admin admin
rabbitmqctl set_user_tags admin administrator
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
raabitmqctl list_users
```

### Install rabbitmqadmin

```shell
cp -a /var/lib/rabbitmq/mnesia/rabbit@my-rabbit-plugins-expand/rabbitmq_management-3.6.12/priv/www/cli/rabbitmqadmin /usr/sbin/rabbitmqadmin
chmod +x /usr/sbin/rabbitmqadmin
apt-get  update && apt-get install python -y
```

### 删除指定的mq

```shell
rabbitmqadmin  list queues | grep ingress | awk '{print $2}' | xargs -I qn rabbitmqadmin delete queue name=qn
rabbitmqadmin  list queues
```

