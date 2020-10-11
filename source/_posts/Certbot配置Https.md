---
title: Certbot 配置 HTTPS
date: 2017-11-06T04:04:46.000Z
tags: ['Server', 'Network', 'Security']
categories: [Ops]
---
## 环境准备

- Ubuntu 16.04
- Nginx

## 安装配置Certbot

添加软件源：

```bash
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
```

可以看到 `/etc/apt/sources.list.d` 目录下多了个 `certbot-ubuntu-certbot-xenial.list` 文件，内容如下：
```
deb http://ppa.launchpad.net/certbot/certbot/ubuntu xenial main
# deb-src http://ppa.launchpad.net/certbot/certbot/ubuntu xenial main
```

安装Certbot：
```
sudo apt-get update
sudo apt-get install python-certbot-nginx 
```

使用如下命令自动配置Nginx服务，也可以加上 `certonly` 参数来手动配置nginx。
```
sudo certbot --nginx
```

出现如下文字，说明Certbot读取了nginx配置文件。默认回车选择所有：
```
Which names would you like to activate HTTPS for?
-------------------------------------------------
1: s1mple.info
2: www.s1mple.info
```

选择是否重定位到HTTPS。

```
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
```

配置成功，有效期为90天，过期需要使用 `certbot renew` 更新新的证书。

```
IMPORTANT NOTES:
- Congratulations! Your certificate and chain have been saved at:
  /etc/letsencrypt/live/simple. info/fullchain.pem
  Your key file has been saved at:
  /etc/letsencrypt/live/simple. info/privkey.pem
  Your cert will expire on 2018-02-04. To obtain a new or tweaked
  version of this certificate in the future, simply run certbot again
  with the “certonly" option. To non-interactively renew xall* of
  your certificates, run "certbot renew"
```

现在你可以通过HTTPS访问你的网站了。

我们查看一下Nginx配置文件，文件名对应你的域名，以 `.conf` 结尾，发现新增了如下内容：

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
说明Nginx在监听443端口，并将不是HTTPS的请求重定位到HTTPS。

## 协议原理

**HTTPS(Http over Secure Socket Layer)** 是基于SSL加密的更安全的HTTP协议，默认端口443。SSL握手协议过程如下：


1. 客户端的浏览器向服务器传送客户端SSL协议的版本号，加密算法的种类，产生的随机数，以及其他通讯所需要信息。
2. 服务器向客户端传送SSL协议的版本号，加密算法的种类，随机数以及其他相关信息，同时**服务器还将向客户端传送自己的证书**。
3. 客户端利用服务器传过来的信息**验证服务器的合法性**，服务器的合法性包括：证书是否过期，发行服务器证书的CA是否可靠，发行者证书的公钥能否正确解开服务器证书的“发行者的数字签名”，服务器证书上的域名是否和服务器的实际域名相匹配。
4. 客户端随机产生一个用于后面通讯的“对话密钥”，然后用服务器的公钥（从步骤2中的服务器的证书中获得）对其加密，然后传给服务器。
5. **服务器用私钥解密**“对话密钥”（此处的公钥和私钥是对应的，公钥加密的数据只能用私钥解密，服务器私钥只在服务器端保留），然后用其作为服务器和客户端的“通话密码”加解密通讯。具体的加密算法详见[RSA加密算法](https://zh.wikipedia.org/wiki/RSA%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95)。
6. SSL的握手部分结束，SSL安全通道的数据通讯开始，客户和服务器开始使用对话密钥进行数据通讯，同时进行通讯完整性的检验。

![78ab162bf20b438ca377faa87f4ea2a2_th](0.png)

## 参考

- [Certbot官方文档](https://certbot.eff.org/#ubuntuxenial-nginx)
- [图解SSL/TLS协议 —— 阮一峰](http://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html)
- [RSA加密算法 —— WiKi](https://zh.wikipedia.org/wiki/RSA%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95)