---
title: Docker镜像的导入导出
date: 2019-02-11 16:07:09
categories:
- Docker
tags:
- Docker
---

## 转载：[Docker镜像的导入导出](https://blog.csdn.net/ncdx111/article/details/79878098)

### 导入导出命令介绍
涉及的命令有export、import、save、load

<!--more-->

#### save

命令 

```
[root@deploy deploy]# docker save --help

Usage:	docker save [OPTIONS] IMAGE [IMAGE...]

Save one or more images to a tar archive (streamed to STDOUT by default)

Options:
  -o, --output string   Write to a file, instead of STDOUT
```



示例 

```
docker save -o nginx.tar nginx:latest 
```

或 

```
docker save > nginx.tar nginx:latest 
```

其中-o和>表示输出到文件，`nginx.tar`为目标文件，`nginx:latest`是源镜像名（name:tag）
#### load
命令 

```
[root@deploy deploy]# docker load --help

Usage:	docker load [OPTIONS]

Load an image from a tar archive or STDIN

Options:
  -i, --input string   Read from tar archive file, instead of STDIN
  -q, --quiet          Suppress the load output
```



示例 

```
docker load -i nginx.tar 
```

或 

```
docker load < nginx.tar 
```


其中`-i`和`<`表示从文件输入。会成功导入镜像及相关元数据，包括tag信息
#### export
命令 

```
[root@deploy deploy]# docker load --help

Usage:	docker load [OPTIONS]

Load an image from a tar archive or STDIN

Options:
  -i, --input string   Read from tar archive file, instead of STDIN
  -q, --quiet          Suppress the load output
[root@deploy deploy]# docker export --help

Usage:	docker export [OPTIONS] CONTAINER

Export a container's filesystem as a tar archive

Options:
  -o, --output string   Write to a file, instead of STDOUT
```



示例 

```
docker export -o nginx-test.tar nginx-test 
```



其中`-o`表示输出到文件，`nginx-test.tar`为目标文件，`nginx-test`是源容器名（name）
#### import
命令 

```
[root@deploy deploy]# docker export --help

Usage:	docker export [OPTIONS] CONTAINER

Export a container's filesystem as a tar archive

Options:
  -o, --output string   Write to a file, instead of STDOUT
[root@deploy deploy]# docker import --help

Usage:	docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]

Import the contents from a tarball to create a filesystem image

Options:
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Set commit message for imported image
```



示例 

```
docker import nginx-test.tar nginx:imp 
```


或 

```
cat nginx-test.tar | docker import - nginx:imp
```


### 区别
- export命令导出的tar文件略小于save命令导出的 

  ![image-20190211161754536](https://ws2.sinaimg.cn/large/006tNc79gy1g02k44bwx6j30pm03qq3i.jpg)

- export命令是从容器（container）中导出tar文件，而save命令则是从镜像（images）中导出

- 基于第二点，export导出的文件再import回去时，无法保留镜像所有历史（即每一层layer信息，不熟悉的可以去看Dockerfile），不能进行回滚操作；而save是依据镜像来的，所以导入时可以完整保留下每一层layer信息。如下图所示，nginx:latest是save导出load导入的，nginx:imp是export导出import导入的。 

  ![image-20190211161828238](https://ws3.sinaimg.cn/large/006tNc79gy1g02k4e8lh3j31a20dytd9.jpg)



### 建议
可以依据具体使用场景来选择命令

- 若是只想备份images，使用save、load即可
- 若是在启动容器后，容器内容有变化，需要备份，则使用export、import



