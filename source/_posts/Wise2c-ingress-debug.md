---
title: WiseCloud中调试Ingress
date: 2018-08-23 15:37:31
categories:
- Wise2c
tags:
- Wise2c
---

## 访问确定

### 在集群中运行命令

```
# 查看下是否能正常访问
curl <podIp地址>:<目标服务端口><目标服务路径>
```

### 进入ingress-gateway运行命令

```
# 查看下是否能正常访问
curl 127.0.0.1:<规则端口><规则路径>
```



## Debug

### 主机端口查看

> 在ingress-gateway所在的主机执行以下命令

```
# 28383为配置的ingress端口(非目标服务端口)
# 在ingress-gateway所在的主机执行以下命令
# 端口需要正常监听到

netstat -nalp | grep 28383
tcp        0      0 0.0.0.0:28383           0.0.0.0:*               LISTEN      16233/nginx: worker

```

### 查看ingress-agent日志

- 使用以下命令查看ingress-agent的具体pod名称

  ```
  kubectl get pods -n wisecloud-agent | grep ingress-agent
  ingress-agent-2sctq                   1/1       Running    0          1h
  ```

- 使用以下命令查看日志

  ```
  kubectl -n wisecloud-agent logs -f ingress-agent-2sctq
  ```


### 查看ingress-gateway日志

- 使用以下命令查看ingress-gateway的具体pod名称

  ```
  kubectl get pods -n wisecloud-agent | grep ingress-gateway
  ingress-gateway-bctnc                 1/1       Running    0          55m
  ```

- 使用以下命令查看日志

  ```
  kubectl -n wisecloud-agent logs -f ingress-gateway-bctnc
  ```


### 查看ingress-gateway配置文件

#### 进入ingress-gateway容器

- 使用以下命令查看ingress-gateway的具体pod名称

  ```
  kubectl get pods -n wisecloud-agent | grep ingress-gateway
  ingress-gateway-bctnc                 1/1       Running    0          55m
  ```

- 使用以下命令进入容器

  ```
  kubectl -n wisecloud-agent exec -it <pod的具体名称> bash
  ```

#### 配置文件

- ##### /root/gateway/www/conf/nginx.conf

  存放ingress转发规则配置文件

  ```
  #daemon off;
  user                        root root;
  worker_processes            auto;
  
  worker_cpu_affinity         auto;	# openresty-1.9.15
  #worker_rlimit_nofile        102400; # ???
  
  error_log                   /var/log/ingress-gateway/error.log;
  #pid                         /var/run/nginx.pid;
  
  events {
      use                     epoll;
      worker_connections      1024;
      multi_accept            on;
  }
  
  stream {
  
  }
  
  http {
      server_tokens                   off;
      sendfile                        on;
      tcp_nodelay                     on;
      tcp_nopush                      on;
      keepalive_timeout               0;
      charset                         utf-8;
  
      include                         mime.types;
      default_type                    application/json;
  
      log_format                      main '[$time_local]`$http_x_up_calling_line_id`"$request"`"$http_user_agent"`$staTus`[$remote_addr]`$http_x_log_uid`"$http_referer"`$request_time`$body_bytes_sent`$http_x_forwarded_proto`$http_x_forwarded_for`$http_host`$http_cookie`$upstream_response_time`xd';
      client_header_buffer_size       4k;
      large_client_header_buffers     8 4k;
      server_names_hash_bucket_size   128;
      client_max_body_size            8m;
  
      client_header_timeout           30s;
      client_body_timeout             30s;
      send_timeout                    30s;
      lingering_close                 off;
  
  
  
  
  
  # upstream 详解
  #
  # upstream www.wise2c.com.28383.jpetstore.rule20.up 
  #	   域名：    www.wise2c.com
  #	   端口：	  28383
  #	   规则路径：/jpetstore/
  #	   规则名称：rule20
  #
  # server 10.244.2.131:8080  weight=1  max_fails=1 fail_timeout=10s;
  #	   目标服务pod的IP地址：10.244.2.131
  #      目标服务端口：8080
  #      目标服务权重(weight)：1
  #	   如果有多条代表有多个pod
  	   
  	   
      upstream www.wise2c.com.28383.jpetstore.rule20.up {
  
          server 10.244.2.131:8080  weight=1  max_fails=1 fail_timeout=10s;
          server 10.244.3.228:8080  weight=1  max_fails=1 fail_timeout=10s;
      }
  
  
  
  
  
      server {
          listen           58080;
          server_name      localhost 127.0.0.1;
          access_log       /var/log/ingress-gateway/ip-access.log  main;
          error_log        /var/log/ingress-gateway/admin.log error;
  
          location / {
              root   html;
              index  index.html index.htm;
          }
  
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   html;
          }
  
          #查看后端服务状态
          location /upstream/status {
              access_log off;
              default_type text/plain;
              content_by_lua_block {
                  local hc = require "resty.upstream.healthcheck"
                  ngx.say("Nginx worker PID:", ngx.worker.pid())
                  ngx.print(hc.status_page())
              }
          }
  
          set $redis_host               '127.0.0.1';
          set $redis_port               '56379';
          set $redis_uds                '/var/run/redis/redis.sock';
          set $redis_connect_timeout    10000;
          set $redis_dbid               0;
          set $redis_pool_size          1000;
          set $redis_keepalive_timeout  90000;
  
  
  
          location /ab_admin {
              content_by_lua_file '../admin/ab_action.lua';
          }
      }
  
      lua_shared_dict api_root_sysConfig 1m;
      lua_shared_dict kv_api_root_upstream 100m;
  
  
      server {
  
  
  
          listen                        28383;
          server_name                   www.wise2c.com;
  # ingress访问日志，即页面上访问时/var/log/ingress-gateway/vhost_access.log会有日志
          access_log                    /var/log/ingress-gateway/vhost_access.log  main;
          
  # ingress访问报错日志，即页面上访问报错时/var/log/ingress-gateway/vhost_error.log会有日志
          error_log                     /var/log/ingress-gateway/vhost_error.log;
  
          # for support ssl
  
  
          # for support data zip
  
  
          # for support backend server.
          # 后端default_backend的Web服务器可以通过X-Forwarded-For获取用户真实IP
          #proxy_redirect                  off;
  
          #proxy_next_upstream             error timeout invalid_header http_500 http_502 http_503 http_504;
  
          set $redis_host               '127.0.0.1';
          set $redis_port               '56379';
          set $redis_uds                '/var/run/redis/redis.sock';
          set $redis_connect_timeout    10000;
          set $redis_dbid               0;
          set $redis_pool_size          1000;
          set $redis_keepalive_timeout  90000;
  
          proxy_headers_hash_max_size     51200;
          proxy_headers_hash_bucket_size  6400;
          proxy_set_header                X-Forwarded-For  $remote_addr;
          proxy_set_header                Host             $http_host;
          proxy_set_header                X-Real-IP        $remote_addr;
          proxy_set_header                X-Forwarded-For  $proxy_add_x_forwarded_for;
  
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   html;
          }
  
  
  
  
  
  
  # proxy_pass 介绍
  # http://www.wise2c.com.28383.jpetstore.rule20.up/jpetstore/
  # http://<域名>.<端口>.<规则路径(去掉两个"/")>.<规则名称>.up<目标路径（target_path）>
  
          location /jpetstore/ {
              proxy_pass                http://www.wise2c.com.28383.jpetstore.rule20.up/jpetstore/;
              proxy_connect_timeout     2s;
          }
  
  
  
  
      }
  
  
      #生产环境下on, 开发环境下off
      lua_code_cache on;
      lua_package_path "../?.lua;../lib/?.lua;../lib/lua-resty-core/lib/?.lua;;";
      lua_need_request_body on;
  }
  ```

- ##### /var/log/ingress-gateway/vhost_access.log

  ingress访问日志，即页面上访问时/var/log/ingress-gateway/vhost_access.log会有日志

  ```
  # tail -f /var/log/ingress-gateway/vhost_access.log
  [23/Aug/2018:15:34:11 +0800]`-`"GET /asf-logo-wide.svg HTTP/1.1"`"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"`200`[171.88.20.181]`-`"http://dev-8:28383/tomcat.css"`0.002`27235`-`-`dev-8:28383`-`0.002`xd
  [23/Aug/2018:15:34:11 +0800]`-`"GET /bg-upper.png HTTP/1.1"`"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"`200`[171.88.20.181]`-`"http://dev-8:28383/tomcat.css"`0.012`3103`-`-`dev-8:28383`-`0.012`xd
  [23/Aug/2018:15:34:11 +0800]`-`"GET /bg-button.png HTTP/1.1"`"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"`200`[171.88.20.181]`-`"http://dev-8:28383/tomcat.css"`0.002`713`-`-`dev-8:28383`-`0.002`xd
  [23/Aug/2018:15:34:12 +0800]`-`"GET /examples/ HTTP/1.1"`"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"`200`[171.88.20.181]`-`"http://dev-8:28383/"`0.007`1126`-`-`dev-8:28383`-`0.007`xd
  [23/Aug/2018:15:34:14 +0800]`-`"GET /examples/jsp HTTP/1.1"`"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"`302`[171.88.20.181]`-`"http://dev-8:28383/examples/"`0.002`0`-`-`dev-8:28383`-`0.002`xd
  [23/Aug/2018:15:34:15 +0800]`-`"GET /examples/jsp/ HTTP/1.1"`"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"`200`[171.88.20.181]`-`"http://dev-8:28383/examples/"`0.053`14326`-`-`dev-8:28383`-`0.053`xd
  [23/Aug/2018:15:34:15 +0800]`-`"GET /examples/jsp/images/execute.gif HTTP/1.1"`"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"`200`[171.88.20.181]`-`"http://dev-8:28383/examples/jsp/"`0.021`1242`-`-`dev-8:28383`-`0.021`xd
  [23/Aug/2018:15:34:15 +0800]`-`"GET /examples/jsp/images/code.gif HTTP/1.1"`"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"`200`[171.88.20.181]`-`"http://dev-8:28383/examples/jsp/"`0.044`292`-`-`dev-8:28383`-`0.044`xd
  [23/Aug/2018:15:34:15 +0800]`-`"GET /examples/jsp/images/return.gif HTTP/1.1"`"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"`200`[171.88.20.181]`-`"http://dev-8:28383/examples/jsp/"`0.038`1231`-`-`dev-8:28383`-`0.038`xd
  [23/Aug/2018:15:34:18 +0800]`-`"GET /examples/jsp/jsp2/el/basic-arithmetic.jsp HTTP/1.1"`"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"`200`[171.88.20.181]`-`"http://dev-8:28383/examples/jsp/"`1.144`1509`-`-`dev-8:28383`-`1.144`xd
  ```

- #####  /var/log/ingress-gateway/vhost_error.log

  ingress访问报错日志，即页面上访问报错时/var/log/ingress-gateway/vhost_error.log会有日志

  ```
  # tail -f /var/log/ingress-gateway/vhost_error.log
  2018/08/23 15:27:23 [error] 110#110: *251 connect() failed (111: Connection refused) while connecting to upstream, client: 171.88.20.181, server: www.wise2c.com, request: "GET /error/ HTTP/1.1", upstream: "http://10.244.3.228:8443/", host: "dev-8:28383"
  2018/08/23 15:28:22 [error] 108#108: *254 connect() failed (111: Connection refused) while connecting to upstream, client: 171.88.20.181, server: www.wise2c.com, request: "GET /error/ HTTP/1.1", upstream: "http://10.244.2.131:8443/", host: "dev-8:28383"
  2018/08/23 15:28:22 [error] 108#108: *254 connect() failed (111: Connection refused) while connecting to upstream, client: 171.88.20.181, server: www.wise2c.com, request: "GET /error/ HTTP/1.1", upstream: "http://10.244.3.228:8443/", host: "dev-8:28383"
  2018/08/23 15:28:23 [error] 108#108: *257 open() "/root/gateway/www/html/favicon.ico" failed (2: No such file or directory), client: 171.88.20.181, server: www.wise2c.com, request: "GET /favicon.ico HTTP/1.1", host: "dev-8:28383", referrer: "http://dev-8:28383/error/"
  2018/08/23 15:30:02 [error] 110#110: *260 "/root/gateway/www/html/errot/index.html" is not found (2: No such file or directory), client: 171.88.20.181, server: www.wise2c.com, request: "GET /errot/ HTTP/1.1", host: "dev-8:28383"
  2018/08/23 15:30:05 [error] 110#110: *261 "/root/gateway/www/html/errot/index.html" is not found (2: No such file or directory), client: 171.88.20.181, server: www.wise2c.com, request: "GET /errot/ HTTP/1.1", host: "dev-8:28383"
  2018/08/23 15:30:15 [error] 110#110: *262 connect() failed (111: Connection refused) while connecting to upstream, client: 171.88.20.181, server: www.wise2c.com, request: "GET /error/ HTTP/1.1", upstream: "http://10.244.3.228:8443/", host: "dev-8:28383"
  2018/08/23 15:30:15 [error] 110#110: *262 connect() failed (111: Connection refused) while connecting to upstream, client: 171.88.20.181, server: www.wise2c.com, request: "GET /error/ HTTP/1.1", upstream: "http://10.244.2.131:8443/", host: "dev-8:28383"
  2018/08/23 15:30:58 [error] 123#123: *271 "/root/gateway/www/html/examples/index.html" is not found (2: No such file or directory), client: 171.88.20.181, server: www.wise2c.com, request: "GET /examples/ HTTP/1.1", host: "dev-8:28383", referrer: "http://dev-8:28383/error/"
  2018/08/23 15:33:22 [error] 124#124: *274 "/root/gateway/www/html/examples/index.html" is not found (2: No such file or directory), client: 171.88.20.181, server: www.wise2c.com, request: "GET /examples/ HTTP/1.1", host: "dev-8:28383", referrer: "http://dev-8:28383/error/"
  ```