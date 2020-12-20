---
title: 部署 Ghost Blog
date: 2017-11-01T05:08:37.000Z
tags: ['Blog Engine']
categories: [Ops]
---
## Preface

> Ghost是比较简约、轻量级的博客平台，而且文档编写使用markdown完成的。所以本片博客将介绍如何在服务器上搭建了Ghost。

## Prerequisites

* Ubuntu 16.04
* MySQL
* NGINX (minimum of 1.9.5 for SSL)
* Systemd
* Node v6 installed via NodeSource
* At least 1GB memory (swap can be used)
* A non-root user for running ghost commands

## Install

[官网安装文档](https://docs.ghost.org/v1.0.0/docs/install)   

> 如果你的数据库在其他服务器上，可以不安装mysql

- 更新软件包源以及更新通过包源安装的软件
    ```
    sudo apt-get update
    sudo apt-get upgrade
    ```
- mysql、ngnix、systemd都可以通过`sudo apt-get install`命令安装
- 由于ghost是基于nodejs开发的，所以要安装nodejs，执行下面这俩条命令
    ```
    curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash 
    sudo apt-get install -y nodejs
    ```
- 安装ghost-cli命令行程序
    ```
    sudo npm i -g ghost-cli
    ```
- 安装完成后可以使用`ghost`命令，`ghost help`查看帮助，ghost安装需要一个空的文件夹
    ```
    mkdir /home/s1mple/ghost
    ```
- `cd`到这个目录下，执行`ghost install`命令即可

> 1.安装过程中会配置数据库，需要提前建个空的database。如果mysql装在本机，就输入localhost，装在别的服务器上就需要输入服务器IP
> 2.询问是否使用nginx可以选no，可以自己手动配置nginx。实际上他帮你配置将80端口的请求转发到ghost的2368端口
> 3.询问是否新建"ghost"用户可以选no，若选yes他会在数据库中新建此用户，并且在ghost后台管理可以看到此用户和一些默认文章。

### After Installation

* 安装后当前文件下会生成**config.production.json**文件，配置信息在这里。`ghost ls`查看当前ghost运行实例信息，`ghost restart`重启ghost。
* 安装完成后，访问`／ghost`进入后台管理系统，根据引导创建用户。
* 在后台Design模块中可以导入主题zip压缩包，`/ghost/#/settings/design`。在Ghost Marketplace下找了个时间轴的主题，并修改了部分视觉效果。[时间轴主题下载链接](https://github.com/s1mplecc/ghost-theme)

### Redirect Port With Nginx

> 很尴尬，因为Aliyun的服务器需要备案，80端口有时候会出现抽风的情况。所以有必要先配置个其他端口（2368端口太难记了），比如8080。

- `cat /etc/nginx/nginx.conf `发现nginx配置文件包括如下两行
    ```
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
    ```
    意思就是配置文件会导入上述两个目录中的文件。所以我们可以`cd /etc/nginx/sites-enabled/`中。
- 发现目录下有个**s1mple.info.conf**文件，这其实就是ghost配置nginx的文件。
    ```
    cp s1mple.info.conf ghost_8080.conf
    ```
- 修改`ghost_8080.conf`中的监听端口80为8080,`service nginx restart`重启nginx
- 修改ghost安装目录下的**config.production.json**,重启ghost
    ```
    "url": "http://s1mple.info:8080",
    ```

> 有个端口号看着很丑，等备案完成，删掉`ghost_8080.conf`就行了。（你看，国外主机就不用备案）
   

## Postscript

* 本来想在docker容器中部署Ghost,但是可能由于docker和宿主系统隔离的原因，systemd命令不能正常运行`Failed to connect to bus: No such file or directory`。暂时没找到解决方法。不过我在docker里部署了另一个博客平台jPress。[跳转链接](https://jpress.s2mple.xyz/)
* jPress是基于java的轻量级博客平台，前身是WordPress。也许下个博客我会介绍如何用docker部署jPress，谁知道呢。







    


