---
title: Dockerfile编写最佳实践
copyright: true
date: 2018-05-29 15:27:01
categories:
- Docker
tags:
- Docker
---

# Dockerfile编写最佳实践

> 转载：https://www.jianshu.com/p/9f15b81759bd
>
> 原文地址：[https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)

Docker 可以从 `Dockerfile` 中读取指令自动构建镜像，`Dockerfile`是一个包含构建指定镜像所有命令的文本文件。Docker坚持使用特定的格式并且使用特定的命令。你可以在 [Dockerfile参考](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/) 页面学习基本知识。如果你刚接触Dockerfile 你应该从哪里开始学习。
<!--more-->
这个文档囊括了`Docker`公司和`Docker`社区推荐的创建易于使用且实用的`Dockerfile` 的最佳实践和方法。我们强烈建议你遵循这些规范（事实上，如果你创建一个官方镜像，你必须坚持这些实践。）

你可以从 [ buildpack-deps ](https://link.jianshu.com?t=https://github.com/docker-library/buildpack-deps/blob/master/jessie/Dockerfile) Dockerifle看到许多这种实践和建议。

> 注：本文档提到的Dockerfile命令的更详细的解释见[Dockerfile参考](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/) 页面。

## [通用参考和建议](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#general-guidelines-and-recommendations)

### [容器应该是临时性的](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#containers-should-be-ephemeral)

从你的Dockerfile定义的镜像启动的容器应该尽可能短暂。这里的『短暂』我们是说它可以被停止和销毁并且一个新容器的构建和替换可以绝对最小化的变更和配置下完成。你可能想看下 应用方法论的12个事实中[进程](https://link.jianshu.com?t=https://12factor.net/processes) 一节来了解以无状态方式运行容器的动机。

### [使用 `.dockerignore`文件](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#use-a-dockerignore-file)

在大多数情况下，最好把`Dockerfile`放在一个空目录里。然后，只把构建`Dockerfile`需要的文件追加到该目录中。为了改进构建性能，你也可以增加一个`.dockerignore` 文件来排除文件和目录。该文件支持与 `.gitignore` 类似的排除模式。更多创建`.dockerignore`信息，见 [.dockerignore](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#dockerignore-file)

### [避免安装不需要的包](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#avoid-installing-unnecessary-packages)

为了减少复杂性，依赖，文件大小，和构建时间，你应该避免仅仅因为他们很好用而安装一些额外或者不必要的包。例如，你不需要在一个数据库镜像中包含一个文本编辑器。

### [每个容器只关心一个问题](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#each-container-should-have-only-one-concern)

解耦应用为多个容器使水平扩容和复用容器更容易。例如，一个web应用栈会包含3个独立的容器，每个都有自己独立的镜像，以解耦的方式来管理web应用，数据库。

你可能听说过"一个容器一个进程"。这种说法有很好的意图，一个容器应该有一个操作系统进程并非真的必要。除此之外，事实上现在容器可以 [被init进程启动](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/run/#/specifying-an-init-process), 一些程序可能会自己产生其他额外的进程。例如，[Celery](https://link.jianshu.com?t=http://www.celeryproject.org/) 可以产生多个工作进程，或者 [Apache](https://link.jianshu.com?t=https://httpd.apache.org/) 可能为每个请求创建一个进程。当然"一个容器一个进程"通常是一个很好的经验法则，**？？但它不是一个很难和快速的规则（it is not a hard and fast rule）？？** 用你最好的判断来保持容器尽可能的干净和模块化。

如果容器之间相关依赖，你可以使用 [Docker容器网络](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/networking/) 来取吧哦容器之间可以通信。

### [最小化层数](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#minimize-the-number-of-layers)

你需要在Dockerfile可读性（从而可以长时间维护）和它用的层数最小化之间找到平衡。**Be strategic 关注你使用的层数（and cautious about the number of layers you use）.**

### [对多行参数排序](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#sort-multi-line-arguments)

无论何时，以排序多行参数来缓解以后的变化（Whenever possible, ease later changes by sorting multi-line arguments alphanumerically. ）。这将帮助你避免重复的包并且使里列表更容易更新。这也使得PR更容易阅读和审查。在反斜线（\）前加一个空格也很有帮助。

这里有个来自 [buildpack-deps 镜像](https://link.jianshu.com?t=https://github.com/docker-library/buildpack-deps)的实例：

```
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```

### [构建缓存](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#build-cache)

在构建镜像的过程中，Docker会逐句读取你Dockerfile中的指令按指定的顺序执行。因为每个指令都会被检查Docker会在它的缓存中查找可以重用的现有镜像（As each instruction is examined Docker will look for an existing image in its cache that it can reuse），而不是创建一个新的（重复的）镜像。如果你根本不像使用缓存，你可以对 `docker build` 命令使用 `--no-cache=ture`参数。

然而，如果你使Docker使用缓存，那么理解它什么时候找到一个匹配的镜像以及什么不找就非常重要了。Docker将遵循的基本规则如下：

- 以一个已经在缓存中的付镜像开始，下一个指令与所有源自该基础镜像的子镜像做对比，来查看镜像中是否有一个使用了完全相同的镜像构建。如果没有，缓存不可用。
- 大多数情况下简单对比Dockfile中的指令与子镜像就足够了。然而，一些特定的指令需要更多的检查和解释。
- 比如`ADD`和`COPY`指令，镜像中的文件内容被检查并且为每个文件计算校验和。这些文件的最终修改和访问时间将不被考虑到校验和内。在查找缓存期间，校验和将被用于与已存在的镜像校验和进行对比。如果文件中有任何变化，比如内容或者元数据，那么缓存失效。
- 除了`ADD`和`COPY`命令以外，缓存检查将不会检查容器中的文件来确定缓存匹配。比如，当处理一个`RUN apt-get -y update`容器中的文件更新将不会被检查来确定是否命中已存在缓存。在这种情况下只有命令字符串自己将被用来查找匹配。

一旦缓存失效，所有的后面的Dockerfile命令将会生成新的镜像而且不会使用缓存。

## [Dcokerfile指令](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#the-dockerfile-instructions)

下面你会找到写Dockerfile里可用的各种指令的建议以及最佳方法。

### [FROM](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#from)

[Dockerfile参考之FROM指令](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#from)

无论何时只要可能使用当前官方仓库镜像作为你的基础镜像。我们推荐[Debian镜像](https://link.jianshu.com?t=https://hub.docker.com/_/debian/), 因为它被严格控制并且保持最小（目前小于150mb），同时是一个完整的发行版。

### [LABEL](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#label)

[理解labels对象](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/labels-custom-metadata/)

你可以给你的镜像增加标签(labels)来协助通过项目组织镜像，记录授权信息，帮助自动化，或者其他原因。每一个标签都以`LABEL`开头并且跟着一对或多对键值对。以下实例展示了可接受的不同格式。解释性意见也包括在内（Explanatory comments are included inline.）。

> 注：如果你的字符串包含空格，它必须被引号引起来或者空格必须被转义。如果你的字符串包含内部引号字符(")，他们需要转义。

```
# Set one or more individual labels
LABEL com.example.version="0.0.1-beta"
LABEL vendor="ACME Incorporated"
LABEL com.example.release-date="2015-02-12"
LABEL com.example.version.is-production=""

# Set multiple labels on one line
LABEL com.example.version="0.0.1-beta" com.example.release-date="2015-02-12"

# Set multiple labels at once, using line-continuation characters to break long lines
LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"
```

查看 [理解labels对象](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/labels-custom-metadata/) 获取可接受的标签键和值指导。
 For information about querying labels, refer to the items related to filtering in [Managing labels on objects](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/labels-custom-metadata/#managing-labels-on-objects).

### [RUN](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#run)

[Dockerfile参考 之 RUN 指令](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#run)

跟之前一样，为了让你的`Dockerfile`具有更高的可读性，更易于理解和维护，使用反斜线()将较长的或者复杂的RUN语句拆分为多行。

#### APT-GET

可能`RUN`最常见的使用场景就是`apt-get`的应用程序了。`RUN apt-get`命令，因为使用它安装软件包有几个需要注意的问题。

你应该避免使用`RUN apt-get upgrade`或者`dis-upgrade`, 因为父镜像中许多"基本的"(essential)包不能在容器中升级。如果父镜像中有个软件包过期了，你应该联系它的维护者。如果你知道有个特定的软件包，`foo`，需要升级，使用`apt-get install -y foo`来自动升级。

通常把`RUN apt-get update` 和 `apt-get install`合并到一个相同的`RUN`语句中，例如：

```
RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo
```

在一个`RUN`语句中单独试用`apt-get update`会引起缓存问题并且导致后面的`apt-get install`指令执行失败。例如，你现在有个`Dockerfile`:

```
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get install -y curl
```

镜像构建完成以后，所有的层都在Docker缓存中。假设你后来修改`apt-get install`增加了其他的软件包：

```
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get install -y curl nginx
```

Docker将最初的指令和修改后的指令视为相同的指令(指apt-get update这行)并且使用上一步的缓存。结果就是`apt-get update`没有执行因为使用了缓存的版本进行构建。因为`apt-get update`没有执行，你的构建可能会安装一个过时版本的`curl`和`ngin`。

使用`RUN apt-get update && apt-get install -y`可以确保你的`Dockerfile`安装最新版本的软件包而无需编码或手动干预。这个技巧被称为"缓存破解"。你也可以通过指定软件包版本来破解缓存。这被称为固定版本，例如：

```
RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo=1.3.*
```

固定版本在构建时强制查找指定版本的软件包而不管缓存有什么。这个技巧可以减少因为依赖包的未知变更导致的失败。

下面是一个格式规范的`RUN`指令，实践了`apt-get`的所有建议。

```
RUN apt-get update && apt-get install -y \
    aufs-tools \
    automake \
    build-essential \
    curl \
    dpkg-sig \
    libcap-dev \
    libsqlite3-dev \
    mercurial \
    reprepro \
    ruby1.9.1 \
    ruby1.9.1-dev \
    s3cmd=1.1.* \
 && rm -rf /var/lib/apt/lists/*
```

`s3cmd`指令指定了版本`1.1.*`。 如果前一个镜像使用了一个老版本，指定新版本会引起`apt-get update`的缓存破解以确保安装新版本。每行列出一个软件包可以避免包重复错误。

另外，你可以通过删除 `/var/lib/apt/lists` 清理apt缓存来减小镜像大小，因为apt缓存不会保存在层里。由于RUN语句以`apt-get update`开头，所以在缓存`apt-get`之前，包缓存将始终被刷新。

> 注：Debian和Ubuntu的镜像自动运行`apt-get clean`，所以不需要显式调用。

#### USING PIPES

一些`RUN`命令依赖使用管道符号（|）把一个命令的输出到另外一个命令的能力，比如以下实例：

```
RUN wget -O - https://some.site | wc -l > /number
```

Docker试用`/bin/sh -c`解释器执行这些命令，它只计算管道最后一个操作的退出代码来确定是否成功。在上面这个例子中只要`wc -l`命令执行成功这一步就构建成功并且生成一个新的镜像，即使`wget`命令失败也是如此。

如果你想让管道中出现任意错误命令都返回错误，在命令前加上`set -o pipefail &&`来确保避免出现未知错误时镜像也能构建成功。例如：

```
RUN set -o pipefail && wget -O - https://some.site | wc -l > /number
```

> 注：并非所有的shell都支持`-o pipefaile`选项。在这种情况下（比如`dash` shell, 它是基于Debian镜像的默认shell），考虑使用`RUN`的exec形式来显式选择一个支持pipefail选项的shell。例如：

```
RUN ["/bin/bash", "-c", "set -o pipefail && wget -O - https://some.site | wc -l > /number"]
```

### [CMD](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#cmd)

[Dockerfile参考 之 CMD指令](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#cmd)

`CMD` 指令用于运行你镜像包含中的软件，连同任意参数。`CMD`应该尽可能都是用这种形式 `CMD [“executable”, “param1”, “param2”…]`。然而，如果是一个作为服务的镜像，比如Apache和Rails,你应该像这样执行`CMD ["apache2","-DFOREGROUND"]`。实际上，实际上，这种形式的指令是推荐用于任何基于服务的镜像。

在其他大多数情况下，`CMD`应该给一个交互式Shell，比如bash，python 和 perl。例如，`CMD ["perl", "-de0"]`, `CMD ["python"]`， 或者 `CMD [“php”, “-a”]`。试用这种形式就意味着当你执行类似`docker run -it python`的一些东西，你将得到一个可用的shell（you’ll get dropped into a usable shell, ready to go）。`CMD`应该很少以`CMD [“param”, “param”]`的形式和`ENTRYPOINT`一起试用，除非你和你的目标用户已经非常熟悉`ENTRYPOINT`工作原理。

### [EXPOSE](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#expose)

[Dockerfile参考之 EXPOSE 指令](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#expose)

`EXPOSE`指令指示容器将监听链接的端口。因此，你应该为你的应用程序试用通用的传统的端口。例如，一个包含Apache Web服务器的镜像应该`EXPOSE 80`, 而一个包含MongoDB的镜像应该使用`EXPOSE 27017`等。

对于外部访问，您的用户可以使用指示如何将指定端口映射到所选端口的标志来执行`docker run`。
 ???For container linking, Docker provides environment variables for the path from the recipient container back to the source (ie, MYSQL_PORT_3306_TCP).???

### [ENV](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#env)

[Dockerfile参考 之 ENV指令](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#env)

为了让软件更便于运行，你可以使用`ENV`来修改环境变量将软件安装目录加到`PATH`。例如：`ENV PATH /usr/local/nginx/bin:$PATH`将使 `CMD [“nginx”]` 可以工作。

`ENV`指令也可用于给要容器化的服务所需的环境变量，比如Postgre的`PGDATA`。

最后，`ENV`也可用于指定通用版本号，这样版本易于维护，如下实例所示：

```
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

和程序中的常量变量类似（和硬编码值相反），这种方法让你可以修改一个单独的`ENV`指令在容器中自动更新容器中的软件版本。

### [ADD or COPY](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#add-or-copy)

[Dockerfile参考之 ADD指令](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#add)
 [Dockerfile参考之 COPY指令](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#copy)

尽管`ADD`和`COPY`指令功能相似，一般而言，最好使用`COPY`。是因为它比`ADD`更透明。`COPY`只支持最基本的从本地复制文件到容器中，而`ADD`有更多功能(比如本地tar解压和远程URL支持)并不是即刻课件的。因此，用`ADD`最好的方式是本地tar文件自动提取到镜像，比如：`ADD rootfs.tar.xz /`。

如果你有多个`Dockerfile`步骤在你的上下文使用不同的文件，单独`COPY`他们，而不是一次复制所有。这将确保每一步的构建缓存（强制这一步重新运行）只有当它特定的依赖文件变化时失效。

例如：

```
COPY requirements.txt /tmp/
RUN pip install --requirement /tmp/requirements.txt
COPY . /tmp/
```

结论就是如果把`COPY . /tmp/`放在`RUN`之前失效缓存更少。

因为镜像大小很重要，使用`ADD`来获取远程URLs是强烈反对的；你应该使用`curl`和`wget`替代。这种方式你可以在解压后不需要时删除这些文件并且你不会在你的镜像增加额外一层。例如，你应该避免这么做：

```
ADD http://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```

并且以此种方式替代:

```
RUN mkdir -p /usr/src/things \
    && curl -SL http://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```

对于不需要ADD tar自动提取功能的其他项目（文件，目录），应始终使用COPY。

### [ENTRYPOINT](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#entrypoint)

[Dockerfile参考 之 ENTRYPOINT指令](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#entrypoint)

使用`ENTRYPOINT`最好的方式是设置镜像主命令，允许镜像把它作为命令运行（然后使用`CMD`作为默认标识）。

我们从一个命令行工具`s3cmd`镜像的例子开始：

```
ENTRYPOINT ["s3cmd"]
CMD ["--help"]
```

现在可以像这样运行镜像来显示命令的帮助：

```
$ docker run s3cmd
```

或者使用正确的参数来执行一个命令：

```
$ docker run s3cmd ls s3://mybucket
```

这样有用，因为镜像名称可以复用为二进制文件的引用，如上面命令所示。

ENTRYPOINT指令也可以与辅助脚本组合使用，允许其以类似于上述命令的方式运行，即使启动工具可能需要多于一个步骤。

例如，[Postgres官方镜像](https://link.jianshu.com?t=https://hub.docker.com/_/postgres/) 使用以下脚本作为它的 `ENTRYPOINT`:

```
#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

> 注：这个脚本使用`exec` [Bash命令](https://link.jianshu.com?t=http://wiki.bash-hackers.org/commands/builtin/exec) 以便最终运行的应用程序成为容器PID 1。这样做允许应用程序接受发送给容器的Unix信号。查看[ENTRYPOINT](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#entrypoint)帮助获取更多细节。

帮助脚本被拷贝到容器并且当容器启动时通过`ENTRYPOINT`运行。

```
COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
```

此脚本允许用户以多种方式与Postgres进行交互。

它可以简单的启动Postgres:

```
$ docker run postgres
```

或者，可以运行Postgres并且传递参数给服务器：

```
$ docker run postgres postgres --help
```

最后，它也可以被用于启动一个完全不同的工具，比如Bash:

```
$ docker run --rm -it postgres bash
```

### [VOLUME](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#volume)

[Dockerfile参考 之 VOLUME指令](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#volume)

`VOLUME`指令应该用于暴露任意数据库存储区，配置存储，或者docker容器创建的文件/目录等。强烈建议您将VOLUME用于镜像的任何可变和/或用户可维护的部分。

### [USER](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#user)

[Dockerfile参考 之 USER指令](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#user)

如果服务可以没有权限运行，使用`USER`变为一个非root用户。像如下命令一样开始在`Dockerfile`中创建用户和组：

```
RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres
```

> 注：**？？？ (Users and groups in an image get a non-deterministic UID/GID in that the “next” UID/GID gets assigned regardless of image rebuilds. )？？？**所以，如果很重要的话，你需要显式指定UID/GID。

> 注：由于Go存档/ tar包处理稀疏文件中的一个[未解决的bug](https://link.jianshu.com?t=https://github.com/golang/go/issues/13548), 在docker容器里创建一个UID足够大的用户会在容器层中将`/var/log/faillog`写满`NUL (\0)`而导致磁盘耗尽。传`--no-log--init`标记来创建用户可以绕开这个问题。Debian/Ubuntu的`adduser`包不支持`--no-log-init`标记所以应该避免使用。

你应该避免安装和使用`sudo`，因为它不可预知的TTY和信号转发行为带来的问题比解决的问题多。如果你确实需要类似`sudo`的功能（例如：以root用户初始化但是以非root用户运行），你可以使用"[gosu](https://link.jianshu.com?t=https://github.com/tianon/gosu)"。

最后，减少你的层和复杂性，避免切换用户（Lastly, to reduce layers and complexity, avoid switching USER back and forth frequently.）。

### [WORKDIR](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#workdir)

[Dockerfile参考之 WORKDIR](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#workdir)

为了清晰可靠，你应该在使用`WORDDIR`时应该一直使用绝对路径。你也应该使用`WORKDIR`而不是使用像`RUN cd .. && do-something`这样难以阅读、调错和维护的增量指令。

### [ONBUILD](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#onbuild)

[Dockerfile参考 之 ONBUILD指令](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/#onbuild)

`ONBUILD`命令在当前`Dockerfile`构建完成之后执行。`ONBUILD`会在任意一个从当前镜像派生的子镜像执行。可以把`ONBUOLD`命令想象成为一个父级`Dockerfile`赋予子`Dockerfile`的指令。

Docker构建在子`Dockerfile`中的任何命令之前执行`ONBUILD`命令。

ONBUILD is useful for images that are going to be built FROM a given image. For example, you would use ONBUILD for a language stack image that builds arbitrary user software written in that language within the Dockerfile, as you can see in Ruby’s ONBUILD variants.

Images built from ONBUILD should get a separate tag, for example: ruby:1.9-onbuild or ruby:2.0-onbuild.

当在`ONBUILD`中使用`ADD`或者`COPY`时要小心。如果新构建的上下文丢失了增加的资源，"onbuild"的镜像将会严重失败。如上所述，添加单独的标签，允许`Dockerfile`的作者自己选择有助于缓解这种情况。

## [官方仓库实例](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#examples-for-official-repositories)

这些官方仓库有典型的示范(These Official Repositories have exemplary Dockerfiles)：

- [Go](https://link.jianshu.com?t=https://hub.docker.com/_/golang/)
- [Perl](https://link.jianshu.com?t=https://hub.docker.com/_/perl/)
- [Hy](https://link.jianshu.com?t=https://hub.docker.com/_/hylang/)
- [Ruby](https://link.jianshu.com?t=https://hub.docker.com/_/ruby/)

## [其他资源](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#additional-resources)

- [Dockerfile Reference](https://link.jianshu.com?t=https://docs.docker.com/engine/reference/builder/)
- [More about Base Images](https://link.jianshu.com?t=https://docs.docker.com/engine/userguide/eng-image/baseimages/)
- [More about Automated Builds](https://link.jianshu.com?t=https://docs.docker.com/docker-hub/builds/)
- [Guidelines for Creating Official Repositories](https://link.jianshu.com?t=https://docs.docker.com/docker-hub/official_repos/)

 

 

 

 

 
