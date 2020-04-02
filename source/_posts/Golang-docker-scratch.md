---
title: Scratch镜像中运行Golang程序
date: 2020-04-02 23:24:35
categories:
- Golang
tags:
- Golang
---



# 前言

记录一次使用scratch镜像运行Golang程序的build镜像过程以及问题解决。scratch镜像来运行Golang能大大缩减最终生成镜像的大小。



# DEMO代码

网上复制的一段Golang代码

```go
# filename: main.go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"os"
)

func main() {
	resp, err := http.Get("https://www.baidu.com")
	check(err)
	body, err := ioutil.ReadAll(resp.Body)
	check(err)
	fmt.Println(len(body))
}

func check(err error) {
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}
```



# 编写Dockerfile

Docker官方提供了Golang各版本的镜像： [Official Repository - golang](https://hub.docker.com/_/golang/)

```dockerfile
# filename: Dockerfile
FROM golang:alpine as builder
ARG GOOS=linux
ARG GOARCH=amd64
RUN mkdir /build
ADD . /build/
WORKDIR /build

# Set necessary environmet variables needed for our image
ENV GOPROXY=https://goproxy.io \
    CGO_ENABLED=0 \
    GOOS=$GOOS \
    GOARCH=$GOARCH
RUN go build -o $software .

FROM scratch
COPY --from=builder /build/main /main
WORKDIR /images
ENTRYPOINT ["/main"]

```

运行命令生成镜像

```shell
docker build -t demo-goland .
```



Dockerfile中一些注解：

- GOPROXY：加上后使过来编译时下载依赖包更容易

- CGO_ENABLED：CGO_ENABLED=0的情况下，Go采用纯静态编译减少对外部动态链接库依赖。

  相关资料：[CGO_ENABLED环境变量对Go静态编译机制的影响](https://johng.cn/cgo-enabled-affect-go-static-compile/)

- GOOS 和 GOARCH：运行的系统平台与架构，Docker中运行Golang请配置`GOOS=linux`。

  相关资料：[Go跨平台编译](https://studygolang.com/articles/16633)



# 可能遇到的问题

### standard_init_linux.go:207: exec user process caused "no such file or directory"

Golang编译后生成的执行文件有依赖外部链接库，在编译的时候加上环境变量`CGO_ENABLED=0`即可解决。

相关参考：[“no such file or directory” with docker scratch image](https://stackoverflow.com/questions/55106186/no-such-file-or-directory-with-docker-scratch-image)

​				   [基于 scratch 镜像构建 docker 镜像报错 ](https://github.com/eddycjy/go-gin-example/issues/90)



### x509: certificate signed by unknown authority

这是一个非常常见的问题：为了进行SSL请求，我们需要SSL根证书。然后scratch镜像是个空镜像，故需要下载证书添加到镜像中。

首先从网站`https://curl.haxx.se/ca/cacert.pem`下载证书，在Dockerfile中添加到目录**/etc/ssl/certs/cacert.pem** (PS: 根据操作系统，这些证书可以在许多不同的地方)

```
# Dockerfile
.....
ADD cacert.pem /etc/ssl/certs/
.....
```



