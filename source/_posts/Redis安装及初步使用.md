---
title: Redis 安装及初步使用
date: 2018-03-15T03:04:12.000Z
tags: ['Redis', 'Server', 'Database']
categories: [Ops]
---
## Preface

> **Redis** 是一个开源的使用 ANSI C 语言编写的高性能 **Key-Value** 数据库，是一个**可基于内存亦可持久化的日志型**的 **NoSQL** 数据库。它将数据保存在内存中，在项目中可用于保存编码表、Token 等**少量数据存储，高速读写访问**的场景

## Installing

- 我的服务器系统为 Ubuntu 16.04 ，直接使用 **apt** 安装。当然也可以在[官网](https://redis.io/download)下载压缩包解压安装

    ```
    sudo apt install -y redis-server
    ```
    
- Redis 的命令主要有两个，一个是服务端命令`redis-server`，用于启动 Redis 。还有个客户端命令`redis-cli`，可用于访问 Redis

    ```
    redis-server -v    // 查看版本
    redis-server -h    // 查看帮助
    ```
    
- 安装完成后 redis-server 已经启动了，默认端口号 **6379**     
    
    ```
    $ ps -ef | grep redis
    redis    14689     1  0 10:43 ?     00:00:00 /usr/bin/redis-server 127.0.0.1:6379
    ```
    
- 注意到 Redis 默认启动时只允许本机（即 127.0.0.1）访问，如果要允许其他主机访问或使用 **GUI** 图形化管理工具，可以修改配置文件`/etc/redis/redis.conf`

    ```
    bind 127.0.0.1 => bind 0.0.0.0
    ```
    如果只需某几个 IP 访问 Redis 服务器，可以修改为对应 IP 以增强安全性
    
- 修改完成后重启 Redis

    ```
    service redis-server restart
    ```
    使用`service redis-server status`命令可以查看 Redis 状态

## redis-cli

> `redis-cli`是客户端程序，可以使用一些命令操作 Redis 数据库，具体命令参考 [Redis 官网](https://redis.io/commands)，或者 [Redis 命令参考](http://doc.redisfans.com/)

- 直接输入`redis-cli`进入客户端，设置一个键值对

    ```
    $ redis-cli
    127.0.0.1:6379> set foo "bar"
    OK
    127.0.0.1:6379> get foo
    "bar"
    ```
    
- `redis-cli`也可查看 Redis 的配置，使用`config get`，命令大小写不敏感，可以使用`TAB`补全

    ```
    127.0.0.1:6379> CONFIG GET bind
    1) "bind"
    2) "0.0.0.0"
    ```
    
- 也可以修改配置，比如我们可以设置 Redis 数据库的密码，默认是不设密码的

    ```
    127.0.0.1:6379> CONFIG SET requirepass passwd
    OK
    
    127.0.0.1:6379> CONFIG GET requirepass
    (error) NOAUTH Authentication required.
    
    127.0.0.1:6379> AUTH passwd
    OK
    
    127.0.0.1:6379> CONFIG GET requirepass
    1) "requirepass"
    2) "passwd"
    ```
    修改密码的格式为`CONFIG SET requirepass password`，修改后需要先登录认证，命令为`AUTH password`

## GUI

> Mac 下我使用的 Redis 图形化管理工具是 **Redis Desktop Manager**

- 输入配置信息连接 Redis 服务器
![99B23802-7EA6-4ECE-9A1F-A63F31624FA0](/0.png)

- 连接上后查看我们刚才设置的键值对
![92EF26D6-3278-43C7-BB81-C34D0D6E6DC7](/1.png)

## References

- [Redis 官网](https://redis.io/)
- [Redis 命令参考](http://doc.redisfans.com/)
- [Redis 中文网](http://www.redis.net.cn/tutorial/3501.html)
