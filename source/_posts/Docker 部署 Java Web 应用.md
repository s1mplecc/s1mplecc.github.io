---
title: Docker 部署 Java Web 应用
date: 2017-11-04T05:42:30.000Z
tags: [Docker, Web]
categories: [Ops]
---
## 前言

JPress是基于JAVA的博客平台，作为一个Web程序，我将用它演示如何构建自己的Docker镜像，以及如何使用Docker部署Java Web应用。

## 准备工作

- 服务器 ubuntu 16.04
- 下载好的jpress.war包，[下载地址](http://jpress.cnitti.net/c/11.html)，下载下来先在IDE里跑了测试一下，以免服务器上出问题
- 构建好的java-web开发环境Docker镜像，[如何构建](http://s1mple.info/deploy-with-docker/)
- Nginx
- Mysql

## 构建JPress镜像

在部署JPress之前我们需要先构建JPress镜像，以便生成容器实例。首先修改Tomcat的配置文件，以适用于JPress。

### server.xml

创建`java-web-env`容器实例，进入bash：

```
docker run -it s1mple1995/java-web-env bash
```

修改Tomcat配置文件：

```
vim /usr/local/lib/tomcat9/conf/server.xml
```

修改8080端口为1110，并修改`<Context>`标签，意思是Web应用文件夹名为`jpress`，访问路径为根路径：

```
<Context docBase="jpress" path="" reloadable="true" />
```

退出容器，并生成新的镜像：

```
docker commit ${container ID} s1mple1995/jpress
```
    
现在你有了jpress的镜像，但是Tomcat的webapps目录下空空如也，我们需要通过**Dockerfile**来构建更完善的jpress镜像。

### Dockerfile

先在服务器上创建一个jpress文件夹，将**jpress.war**`scp`到此目录下
```
mkdir -p /home/s1mple/jpress
```
执行`vim Dockerfile`编写镜像构建文件
```
# Base image
From s1mple1995/jpress

USER root

# environment variables
ENV APP /usr/local/lib/tomcat9/webapps

# Copy the current directory contents into the container at $APP
ADD ./jpress.war $APP

# Make port 1110 available to the world outside this container
EXPOSE 1110

WORKDIR $APP

CMD ["service", "tomcat", "run"]
```

其中，**From**指依赖的基础镜像。**USER**默认就是root，在生产环境为了安全可以指定其他用户。**ENV**是环境变量。**ADD**或者**COPY**命令将我们的war包拷贝到webapps目录下。**EXPOSE**开放端口。**WORKDIR**表示你进入容器时的初始路径。**CMD**为容器启动时执行的命令。这里使用`service tomcat run`而不是`start`是因为容器会监控前台如果没有输出，在`start`命令执行完后就会自动退出。而`run`命令是前台运行。我之前一直用`start`命令发现容器总是刚启动就退出。
更多命令请参考[官方文档](https://docs.docker.com/engine/reference/builder/#from)。


### 构建Docker镜像

现在所有准备工作都已完成，你可以构建自己的JPress镜像了。

在`/home/s1mple/jpress`目录下执行，`-t`给新镜像打上标签：

```
docker build -t s1mple1995/jpress .
```

可以看到控制台输出了一些信息：

```
Sending build context to Docker daemon 20.@6MB
Step 1/6 : FROM simple1995/jpress
---> 6c7F7aa5716d
Step 2/6 : USER root
---> Running in d29bd2d7cf4e
---> 616922ef516c
Removing intermediate container d29bd2d7cf4e
Step 3/6 : ENV APP /usr/local/lib/tomcat9/webapps
---> Running in 5667a207¢898
---> b6c061148616
Removing intermediate container 5667a267¢898
Step 4/6 : ADD ./jpress.war SAPP
---> a82c14e949e0
Step 5/6 : EXPOSE 1110
---> Running in b31964ddfefo
---> cd9de2d7786F
Removing intermediate container b31964ddfefo
Step 6/6 : CMD service tomcat run
---> Running in @4a6a78fa77a
---> de9b3648553e
Removing intermediate container @4a6a78fa77a
Successfully built de9b3648553e
Successfully tagged s1mple1995/jpress:latest
```

Dockerfile里的每一句命令都会在一个新的容器中运行，并生成一个新的镜像，再将中间过渡的容器移除。

我们来验证一下，执行`docker images`看到我的两个镜像：

```
REPOSITORY                  TAG          IMAGE ID
s1mple1995/jpress           latest       de9b3648553e
simple1995/java-web-env     latest       292d98d8d6d4
```

执行`docker image history de9`查看jpress镜像历史

![-----2017-11-04---7.01.30](/2.png)

甚至可以看到我们通过`bash`修改tomcat配置文件后提交的镜像，你可以通过运行这些镜像来回退你的操作

## 运行

运行，`-p`绑定端口，`-v`挂载宿主机与容器的共享文件夹：

```
docker run -it -d -p 1110:1110 -v /home/s1mple/jpress/:/usr/local/lib/tomcat9/webapps s1mple1995/jpress
```

不出意外，你现在就可以在浏览器上访问`http://{domain}:1110/install`进入JPress安装向导，按照步骤，填写数据库信息，需要提前建库。数据库主机填写你安装了mysql的服务器IP。

如果长时间未加载，可以执行`docker restart`命令重启容器

## Nginx重定位

通过nginx配置二级域名，你现在可以通过 http://jpress.s1mple.info 访问我的jpress博客。

首先你要保证开放了二级域名。我的域名是在GoDaddy上买的，需要在他的DNS管理界面添加`*.s1mple.info`。

添加nginx配置文件：

```
cd /etc/nginx/sites-enabled
cp ghost_8080.conf jpress.conf
```

修改jpress.conf文件，添加80、8080端口：

```
server_name jpress.s1mple.info www.jpress.s1mple.info;

location / {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Host $http_host;
    proxy_pass http://127.0.0.1:1110;
}
```

执行`service nginx restart`即可。

效果图
![-----2017-11-20---9.42.47](/3.png)