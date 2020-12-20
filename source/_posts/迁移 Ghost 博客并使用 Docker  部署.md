---
title: 迁移 Ghost 博客并使用 Docker  部署
date: 2018-06-24T14:30:15.000Z
tags: ['Docker','Blog Engine']
categories: [Ops]
---
## Preface

> 因为阿里云服务器在七月初即将过期，以及生活上的一些琐事，博客已经近一个月没有更新了。考虑到十月份域名也将过期，以及阿里云非学生用户太贵，遂考虑国外的云服务器 [Vultr](https://www.vultr.com/)，优点是绑定域名不需要备案，以及按时计费也很方便，缺点是国内访问网速确实比不上阿里云，但同时也跟同学了解到可以使用**锐速**进行优化，这个以后再研究。此外，为将来迁移方便，使用 **Docker** 重新部署 Ghost 博客并制作成镜像上传到云端。这也算我迁移博客后的第一篇文章了。

## Prefaring

- 在 Vultr 的 Servers 界面先新增一个实例，如果能抢到每月 $2.5 的服务器是最好的，然而我没有抢到，所以选择了每月 $5 的服务器，系统为 Ubuntu 16.04，机房在新加坡

- 在添加实例成功后第一件事是`ping`一下公网 IP 是否是通的，有些时候分配的 IP 是被大陆给墙了的。`ping`通后我习惯的首件要事就是添加我的 ssh 公钥以及修改 ssh 的一些策略，比如修改默认端口号、禁用密码登陆之类的，一切都是为安全考虑（吃了第一次的亏，一夜之间异常登陆三十次）

- 迁移 Ghost 博客之前需要先将原先服务器上的相关文件下载到本地，再上传到新的服务器上。主要包含文章内容，图片以及主题。所幸 Ghost 博客提供了 Export/Import 功能，而且是所有文件导出到一个大 Json 文件中，很方便迁移
![14070DA1-C3CD-4A9F-AE52-D4D2DE4A2CDB](/0.png)主题也很方便，我之前就将代码上传至 [GitHub](https://github.com/s1mplecc/ghost-theme-kaldorei)，克隆下来压缩上传就行。图片稍微麻烦点，需要将 Ghost 目录下的整个`content/images`文件夹`scp`到本地，再`scp`到 Vultr 服务器上，可能要花些时间

- 接下来去修改你的域名设置，我是在 [GoDaddy](https://sg.godaddy.com/zh) 上购买的域名，所以去狗爹网修改配置如下
![64F8A358-E544-4FA3-819C-7F38F5327A5A](/1.png)

- 准备工作已经做好，剩下的就是在新服务器上搭建环境和部署博客了。必要的东西也就三个，Nginx 用于做端口映射，Docker 用于运行 Ghost 容器，Certbot 用来配置 Https。Nginx 在 Ubuntu 上安装很简单，使用`apt-get`命令即可。Certbot 安装在我的另一篇博客[Certbot配置Https](https://s2mple.xyz/configure-https-with-certbot/)中有详细介绍。我们着重介绍一下如何安装 Docker

## Installing Docker

> 参考 [Docker官方网站](https://docs.docker.com/install/linux/docker-ce/ubuntu/)，选择自己的操作系统，我的是 Ubuntu 16.04

- Update the apt package index, Install packages to allow apt to use a repository over HTTPS

    ```
    $ sudo apt-get update
    $ sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
    ```
    
- Add Docker’s official GPG key

    ```
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```
    
- Set up the stable repository

    ```
    $ sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"
    ```
    
- Install the latest version of Docker CE

    ```
    $ sudo apt-get update
    $ sudo apt-get install docker-ce
    ```
    
## Deploying Ghost

- 拉取 Ghost 最新的 Docker 镜像

    ```
    $ sudo docker pull ghost
    ```
    
- 运行一个容器实例，`-d`后台运行，`-p 2368:2368`绑定宿主机 2368 端口至容器内 2368 端口（必须显式的绑定，不然不会自动映射），这也是 Ghost 默认占用的端口号

    ```
    $ sudo docker run -d -p 2368:2368 ghost
    ```
    
- 运行后访问`{IP}:2368`可以成功进入 Ghost 主页，但是现在还有一些问题，就是点击 Home 键跳转的地址为 localhost:2368，我们需要进入到容器内部修改 Ghost 配置文件

    ```
    $ sudo docker exec -it 16 bash
    ```
    上述命令表示以交互式终端执行容器的`bash`命令，其中 16 是刚才启动的容器 ID 前两位，可以使用`docker ps`查看，通常使用可以唯一标识的前几位即可
    
- 进入容器后默认在`/var/lib/ghost`即 Ghost 安装路径下，修改其下的`config.production.json`配置文件，其实该文件链接到`config.development.json`，所以修改哪一个都一样，只需要修改 url 和 server 即可

    ```json
    "url": "https://s2mple.xyz/",
    "server": {
        "port": 2368,
        "host": "0.0.0.0"
    },
    ```
    退出容器后使用`docker restart 16`重启容器
    
- 主题可以在网页上直接上传。图片还需要我们拷贝到容器中，使用如下命令

    ```
    $ sudo docker cp images/ 16:/var/lib/ghost/content
    ```
    
## Nginx & Https

> 至此，我们的博客已经算迁移完毕，接下来还剩下两个步骤：1. 配置 Nginx 80 端口转发到 2368 端口；2. 使用 Certbot 配置 Https。

#### Configuring Nginx

- 前往 `/etc/nginx/conf.d/` 目录中创建 ghost.conf 配置文件，配置如下

    ```
    server {                                                                                  
        server_name s1mple.info www.s1mple.info;

        location / {
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $http_host;
            proxy_pass http://127.0.0.1:2368;
        }

        location ~ /.well-known {
            allow all;
        }   

        client_max_body_size 50m;
    }
    ```
    保存退出后，`service nginx restart` 重启 Nginx 服务

#### Configuring Https

- 使用 Certbot 工具配置 Https，如何安装运行可以参考我的另一篇博客[
Certbot配置Https](https://s2mple.xyz/configure-https-with-certbot/)。实际上它会读取 Nginx 配置文件并修改它们，可以发现，配置完成后的 ghost.conf 多了如下一些内容，其中就包括重定位到 Https 端口 443

    ```
    server {
    
        # 省略...

        listen [::]:443 ssl ipv6only=on; # managed by Certbot
        listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/s2mple.xyz/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/s2mple.xyz/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
        
    }
    
    server {
    
        if ($host = www.s1mple.info) {
            return 301 https://$host$request_uri;
        } # managed by Certbot


        if ($host = s1mple.info) {
            return 301 https://$host$request_uri;
        } # managed by Certbot

        listen 80; 
        listen [::]:80;

        server_name s1mple.info www.s1mple.info;
        return 404; # managed by Certbot
    }
    ```
    大功告成！现在就可以通过 https://s2mple.xyz/ 访问我的博客网站了！
    
## Docker image

> 接下来我们将经过修改的容器（包含文章、图片、Kaldorei 主题、修改后的配置文件）提交为一个镜像，上传到云端。以后迁移直接`pull`下来即可。可以参考我之前的博客 [构建你的Docker镜像](https://s2mple.xyz/build-docker-images/)

- 首先需要在 [Docker Cloud官网](https://cloud.docker.com/) 注册自己的账号，然后在服务器使用命令登陆

    ```
    $ sudo docker login
    Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
    // username
    // password
    Login Succeeded
    ```
    
- 在 Docker Cloud 中创建一个新的仓库，命名为 myghost 。将当前容器提交为一个镜像，不加标签默认 latest

    ```
    $ sudo docker commit 16 s1mple1995/myghost
    ```
    
- 查看本地的 Docker 镜像，有两个，一个是原始的 ghost 镜像，一个是刚刚提交的镜像

    ```
    $ sudo docker images
    REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
    s1mple1995/myghost   latest              e11def1af177        15 seconds ago      613MB
    ghost                latest              a375d6b66f06        4 weeks ago         574MB
    ```
    
- 将镜像`push`到云端，等待上传完成

    ```
    $ sudo docker push s1mple1995/myghost
    ```

- 现在镜像已经存放在 Docker Cloud 云端了，搜索你的镜像，随时随地可以`pull`下来

    ```
    $ sudo docker search s1mple1995
    NAME                      DESCRIPTION                    STARS               OFFICIAL            AUTOMATED
    s1mple1995/get-started    学习初步使用                         1                                       
    s1mple1995/java-web-env   Java Web程序的运行环境                0                                       
    s1mple1995/jpress         jpress blog                    0                                       
    s1mple1995/ubuntu         基于官方ubuntu镜像，更改为阿里云软件源，并安…     0                                       
    s1mple1995/myghost        我的 Ghost 镜像，包含了文章、图片和主题（ Ka…   0 
    ```

## Conclusion

> 总的来说，使用 Docker 部署 Ghost 的话可以方便许多。而且制作成镜像上传到云端的话，以后可以直接`pull`下来，图片、文章、主题都在镜像中了，顶多是更换域名要修改 Ghost 的配置文件。除此之外，更换域名后文章中的域名也需要全量替换，这个我打算以后要换域名的时候写个 Python 脚本来完成。
