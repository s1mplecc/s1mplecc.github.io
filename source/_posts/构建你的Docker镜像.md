---
title: 构建你的 Docker 镜像
date: 2017-11-02T05:44:16.000Z
tags: ['Server', 'Docker']
categories: [Ops]
---
## Preface

> Docker实现了宿主系统和Docker容器(container)的隔离，使用Docker可以将构建开发所需环境的镜像(images)，再根据镜像生成容器实例。实现不依赖宿主机环境的高效部署、测试。

油条有一篇很好的文章介绍了Docker的工作原理。[链接](https://0x233.com/shen-qi-de-dockerdao-di-zuo-liao-shi-yao/)

## Get Started

安装Docker,[官网文档](https://docs.docker.com/engine/installation/)

> 1.官方镜像仓库：https://hub.docker.com/
> 2.官网下载速度较慢，推荐DaoCloud仓库，和官网同步更新的 https://hub.daocloud.io/

下载ubuntu基础镜像，冒号后是标签

```
docker pull daocloud.io/library/ubuntu:16.04
```
执行`docker images`查看已下载的镜像

创建容器实例，返回容器ID。`-d`后台运行

```
docker run -d -it daocloud.io/library/ubuntu:16.04
```
执行`docker ps`查看正在运行的容器

## Build Your Image

进入容器，ID输入可区分的几位就可以了

```
docker exec -it 4e bash
```
可以看到在容器中有一套全新的目录结构，与宿主机是隔离的。

更改软件源为阿里云源

```
cd /etc/apt/
cp sources.list sources.list_backup 
cat /dev/null > sources.list 
```

`vim sources.list`将如下内容加入文件

```
deb http://mirrors.cloud.aliyuncs.com/ubuntu/ xenial main
deb-src http://mirrors.cloud.aliyuncs.com/ubuntu/ xenial main
deb http://mirrors.cloud.aliyuncs.com/ubuntu/ xenial-updates main
deb-src http://mirrors.cloud.aliyuncs.com/ubuntu/ xenial-updates main
deb http://mirrors.cloud.aliyuncs.com/ubuntu/ xenial universe
deb-src http://mirrors.cloud.aliyuncs.com/ubuntu/ xenial universe
deb http://mirrors.cloud.aliyuncs.com/ubuntu/ xenial-updates universe
deb-src http://mirrors.cloud.aliyuncs.com/ubuntu/ xenial-updates universe
deb http://mirrors.cloud.aliyuncs.com/ubuntu/ xenial-security main
deb-src http://mirrors.cloud.aliyuncs.com/ubuntu/ xenial-security main
deb http://mirrors.cloud.aliyuncs.com/ubuntu/ xenial-security universe
deb-src http://mirrors.cloud.aliyuncs.com/ubuntu/ xenial-security universe
```

日常`apt-get update|upgrade`之后着手安装jdk和Tomcat了

我的另一篇博客详细介绍了如何[安装 JDK&Tomcat](http://s1mple.info/install-java-tomcat/)

> 如果你成功到了这里，那恭喜你，你自己的开发环境镜像已经构建完了。还需要一些步骤来将你的镜像推上云端！

## Push It !

`exit`或者`ctrl+d`退出当前容器，`docker ps`查看容器ID

根据容器创建一个新的镜像，不加标签默认**latest**

```
docker commit 4e s1mple1995/java-web-env
```

在`push`前需要在[docker官网](https://cloud.docker.com/)注册账户，用于存放自己的镜像

在**Docker Cloud**中**create repository**，镜像名要和服务器中的镜像名一致，不一致可以使用如下命令

```
docker tag local-image:tagname new-repo:tagname
```

在服务器端登录

```
docker login
```

执行push命令，等待上传完成

```
docker push s1mple1995/java-web-env
```

现在你的镜像已经存放在Docker Cloud云端了，搜索你的镜像，随时随地可以pull下来。

```
docker search s1mple1995
```

#### Tips

1. Docker的镜像管理很像git的版本控制。
2. 你也可以使用阿里云的容器镜像服务。如果你的服务器是阿里云的ECS，可以使用内网传输，速度非常快。
  
    ```
    docker login registry-vpc.cn-hangzhou.aliyuncs.com
    ```







​    


