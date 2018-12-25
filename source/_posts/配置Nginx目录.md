---
title: 配置Nginx目录
date: 2017-11-19T02:07:54.000Z
tags: ['Linux', 'Server']
categories: []
---
## Preface

> 配置Nginx目录，可以很方便的下载镜像，查看文件等等

- 效果图
![-----2017-11-19---10.54.27](/0.png)

## Configure Nginx

- 前往nginx配置目录
    ```
    cd /etc/nginx/sites-enabled
    ```

- `vi dir_80.conf`添加配置文件
    ```
    server {
        listen 80;
        listen [::]:80;

        server_name dir.s1mple.info www.dir.s1mple.info;
        root /var/www/html;

        location / {
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $http_host;
            autoindex on;
            autoindex_exact_size off;
        }

        listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/s1mple.info/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/s1mple.info/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

        # Redirect non-https traffic to https
        if ($scheme != "https") {
            return 301 https://$host$request_uri;
        } # managed by Certbot

    }
    ```
    
- `service nginx restart`重启nginx服务

#### Tips

1. 监听80端口，域名为dir.s1mple.info和www.dir.s1mple.info
2. 根目录为/var/www/html
3. **autoindex on**打开目录浏览功能
4. **autoindex_exact_size**默认为on，显示文件确切大小，单位bytes，改为off后显示大概大小，单位kB或者MB或者GB
5. 剩下的是**Certbot**重定位https的配置

## Configure Certbot

- 重新配置Certbot
    ```
    certbot run
    ```
    
- 它会自动检测nginx的配置文件
![-----2017-11-19---10.31.17](/1.png)

- 提示已经存在证书，并且已经配置了一些域名，询问是否扩充，选择E
![-----2017-11-19---10.32.45](/2.png)

- 询问是否将HTTP重定位到HTTPS，选择2是所有请求均重定位到HTTPS
![-----2017-11-19---10.38.24](/3.png)