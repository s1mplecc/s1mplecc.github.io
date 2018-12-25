---
title: 离线安装Nginx
date: 2018-01-22T05:22:55.000Z
tags: ['Server', 'Nginx']
categories: []
---
## Preface

> 工作中难免有不能使用外网的环境，虽然说不能连外网安装软件真的是很费劲，但是为了安全着想也没得办法嘛

---
## Prerequisites

- **nginx-1.12.2.tar.gz** [官网](http://nginx.org/en/download.html)下载tar包，下载链接：http://nginx.org/download/nginx-1.12.2.tar.gz
- **pcre-8.41.tar.gz**
- **zlib-1.2.11.tar.gz**
- **openssl-1.0.2n.tar.gz**
- 配置好 **ssh** 免密钥登录服务器

---
## Installing

- 上传各个 tar 包至服务器的`/opt`目录，然后使用`tar -xzvf`解压

    ```
    scp /Users/s1mple/Downloads/nginx-1.12.2.tar.gz root@10.0.1.16:/opt
    scp /Users/s1mple/Downloads/pcre-8.41.tar.gz root@10.0.1.16:/opt
    scp /Users/s1mple/Downloads/zlib-1.2.11.tar.gz root@10.0.1.16:/opt
    scp /Users/s1mple/Downloads/openssl-1.0.2n.tar.gz root@10.0.1.16:/opt
    ```

- 前往解压出来的`/opt/nginx-1.12.2`文件夹，其中包含了 **Nginx** 的配置脚本`configure`，运行这个脚本并指导刚才解压的各依赖的路径

    ```
    ./configure --with-pcre=/opt/pcre-8.41 --with-zlib=/opt/zlib-1.2.11 --with-openssl=/opt/openssl-1.0.2n
    ```

- 执行后的结果，生成了 **Makefile** 文件，并输出了一些配置的摘要。简单看一下，默认的 **Nginx** 安装在`/usr/local/nginx`下，以及启动脚本、配置、日志的路径

    ```
    creating objs/Makefile

    Configuration summary
      + using PCRE library: /opt/pcre-8.41
      + using OpenSSL library: /opt/openssl-1.0.2n
      + using zlib library: /opt/zlib-1.2.11

      nginx path prefix: "/usr/local/nginx"
      nginx binary file: "/usr/local/nginx/sbin/nginx"
      nginx modules path: "/usr/local/nginx/modules"
      nginx configuration prefix: "/usr/local/nginx/conf"
      nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
      nginx pid file: "/usr/local/nginx/logs/nginx.pid"
      nginx error log file: "/usr/local/nginx/logs/error.log"
      nginx http access log file: "/usr/local/nginx/logs/access.log"
      nginx http client request body temporary files: "client_body_temp"
      nginx http proxy temporary files: "proxy_temp"
      nginx http fastcgi temporary files: "fastcgi_temp"
      nginx http uwsgi temporary files: "uwsgi_temp"
      nginx http scgi temporary files: "scgi_temp"
    ```

- `make`是 **Linux** 下常用的构建工具，主要用于 C 语言，它会根据 **Makefile** 中的构建规则生成文件。在`/opt/nginx-1.12.2`目录下执行这俩个命令

    ```
    make
    make install
    ```
    
- `/opt/nginx-1.12.2`目录下的 **Makefile** 文件包含了实际执行的命令。`-f`指定了刚刚生成的`/objs/Makefile`文件

    ```
    default:    build
    build:      $(MAKE) -f objs/Makefile
    install:    $(MAKE) -f objs/Makefile install
    ```
    
- 构建完成后前往`/usr/local/nginx`查看生成的文件

    ```
    $ ls /usr/local/nginx
    client_body_temp  fastcgi_temp  logs        sbin       uwsgi_temp
    conf              html          proxy_temp  scgi_temp
    ```
    
---
## Running
    
- 运行`sbin/nginx`启动脚本并验证是否已启动

    ```
    $ ./sbin/nginx
    $ ps -ef | grep nginx
    root     31402     1  0 11:48 ?        00:00:00 nginx: master process ./nginx
    nobody   31403 31402  0 11:48 ?        00:00:00 nginx: worker process
    ```
    
- 浏览器访问 10.0.1.16 也已显示 **Nginx** 启动成功
![Screen-Shot-2018-01-22-at-11.50.30-AM](/0.png)
 
---
## Configuring

- 在`/usr/local/nginx/conf`中包含了 **Nginx** 配置文件，我们先做个备份

    ```
    cp nginx.conf nginx.conf.bk
    ```

- 修改`nginx.conf`文件，在`http{}`中加入这一行表示引入外部配置文件

    ```
    include /usr/local/nginx/conf/conf.d/*.conf;
    ```

- 在`/usr/local/nginx/conf`中创建`conf.d`文件夹，并创建自己的配置文件，注意要以`.conf`结尾，添加`server{}`后重启 **Nginx** 即可

    ```
    server {                                                                                
        listen 80;
        listen [::]:80;

        server_name 10.0.1.16;
        root /var/www/dist;

        location / {
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $http_host;
        }
    }
    ```

---
## Postscripts

> 安装下来感觉就是，本来一条命令就解决的事，你比如说 **sudo apt-get install nginx** 结果强行搞这么复杂，网络带来的便利无疑是巨大的，所以说安全方面的意识和技能非常关键

---
## References

- [Nginx中文文档](http://www.nginx.cn/doc/)