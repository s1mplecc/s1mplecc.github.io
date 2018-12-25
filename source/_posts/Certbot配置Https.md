---
title: Certbot配置Https
date: 2017-11-06T04:04:46.000Z
tags: ['Linux', 'Server', 'Network', 'Security']
categories: []
---
## Preface

> 天天看着自己的网站“不安全”很不开心，在油条那儿了解到一个叫做Certbot的工具，可以简单快捷的配置http为https

## Prerequisite

- ubuntu 16.04
- nginx

## Get Started

- [官方文档](https://certbot.eff.org/#ubuntuxenial-nginx)

- 导入软件源

    ```bash
    sudo apt-get update
    sudo apt-get install software-properties-common
    sudo add-apt-repository ppa:certbot/certbot
    ```

#### Tips
> add-apt-repository is a script for adding apt sources.list entries.It can be used to add any repository and also provides a shorthand syntax for adding a Launchpad PPA (Personal Package Archive) repository.

- 可以看到`/etc/apt/sources.list.d`目录下多了个`certbot-ubuntu-certbot-xenial.list`文件，内容如下
    ```
    deb http://ppa.launchpad.net/certbot/certbot/ubuntu xenial main
    # deb-src http://ppa.launchpad.net/certbot/certbot/ubuntu xenial main
    ```

- 安装**Certbot**
    ```
    sudo apt-get update
    sudo apt-get install python-certbot-nginx 
    ```

- 使用如下命令自动配置nginx服务，当然你也可以加上`certonly`参数来手动配置nginx
    ```
    sudo certbot --nginx
    ```
    
- 出现如下文字，说明certbot可以识别nginx配置文件，读取其中信息   
![-----2017-11-06---10.49.41](/0.png)
    直接回车选取所有

- 选2的话所有网页都会被重定位到https
![-----2017-11-06---11.05.39](/1.png)

- 配置成功，可以看到有效期是90天，需要使用`certbot renew`生成新的证书
![-----2017-11-06---11.10.43](/2.png)

- 现在你就可以通过https访问你的网站了

### Nginx Configuration

- 我们看一下nginx配置文件，`vim /etc/nginx/sites-enabled/s1mple.info.conf`，
发现新增了如下内容
   
    ```
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/s1mple.info/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/s1mple.info/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    # Redirect non-https traffic to https
    if ($scheme != "https") {
        return 301 https://$host$request_uri;
    } # managed by Certbot
    ```
    监听443端口，重定位https，301也是重定位的状态码
    
- 执行`netstat -tulpen`也可以看到nginx确实在监听443端口
    ```
    tcp     0.0.0.0:443     0.0.0.0:*    LISTEN    11873/nginx -g daem
    ```
    
## About Https

- [Https协议原理](http://jingyan.baidu.com/article/e4d08ffdb61f040fd3f60d48.html)
- [Web 加速，协议先行！](http://news.ifeng.com/a/20170904/51862764_0.shtml)

> **Https(Http over Secure Socket Layer)** 是基于ssl加密的更安全的http协议，默认端口443。而**Certbot**部署的的**Let's Encrypt**证书就是“让我们加密”的意思。

### SSL握手协议

- [传输层安全协议抓包分析之SSL/TLS](http://www.188cf.net/licai/87869x.shtml)

![78ab162bf20b438ca377faa87f4ea2a2_th](/3.jpeg)

> ① 客户端的浏览器向服务器传送客户端SSL协议的版本号，加密算法的种类，产生的随机数，以及其他通讯所需要信息。
② 服务器向客户端传送SSL协议的版本号，加密算法的种类，随机数以及其他相关信息，同时**服务器还将向客户端传送自己的证书**。
③ 客户端利用服务器传过来的信息**验证服务器的合法性**，服务器的合法性包括：证书是否过期，发行服务器证书的CA 是否可靠，发行者证书的公钥能否正确解开服务器证书的“发行者的数字签名”，服务器证书上的域名是否和服务器的实际域名相匹配。
④ 客户端随机产生一个用于后面通讯的“**对称密钥**”，然后用服务器的公钥（服务器的公钥从步骤②中的服务器的证书中获得）对其加密，然后传给服务器。
⑤ **服务器用私钥解密**“对称密钥”(此处的公钥和私钥是相互关联的，**公钥加密的数据只能用私钥解密**，**私钥只在服务器端保留**。详细请参看： http://zh.wikipedia.org/wiki/RSA%E7%AE%97%E6%B3%95)，然后用其作为服务器和客户端的“**通话密码**”加解密通讯。同时在SSL通讯过程中还要完成数据通讯的完整性，防止数据通讯中的任何变化。
⑥ SSL的握手部分结束，SSL安全通道的数据通讯开始，客户和服务器开始**使用相同的对称密钥进行数据通讯**，同时进行通讯完整性的检验。
    
 