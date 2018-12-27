---
title: 搭建 SSR 服务器 + 锐速
date: 2018-08-30T01:47:13.000Z
tags: ['Server']
categories: [Ops]
---
## Prerequisites

- **服务器**，我是用的是 Vultr，注意购买国外的服务器，因为要用于翻墙
- **系统版本：Ubuntu 16.04**，因为要更换内核，本教程只适用于 Ubuntu 16.04 版本，其他版本或者其他系统请自行查阅如何更换内核
- **ShadowsocksR客户端**，客户端准备好 ShadowsocksR 软件用于连接服务器端，下载地址见[该文章](https://github.com/Alvin9999/new-pac/wiki/%E8%87%AA%E5%BB%BAss%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%95%99%E7%A8%8B)的【SSR客户端下载】部分

## 安装SSR

- 下载并执行 **ShadowsocksR 单/多端口一键管理脚本**

    ```
    wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssr.sh && chmod +x ssr.sh && bash ssr.sh
    ```
    备用命令
    ```
    wget -N --no-check-certificate https://softs.fun/Bash/ssr.sh && chmod +x ssr.sh && bash ssr.sh
    ```
    
- 运行`ssr.sh`脚本后，按照步骤依次设置端口、密码、加密方式等
    
    ```
      ShadowsocksR 一键管理脚本 [v2.0.38]
      ---- Toyo | doub.io/ss-jc42 ----

      1. 安装 ShadowsocksR
      2. 更新 ShadowsocksR
      3. 卸载 ShadowsocksR
      4. 安装 libsodium(chacha20)
    ————————————
      5. 查看 账号信息
      6. 显示 连接信息
      7. 设置 用户配置
      8. 手动 修改配置
      9. 切换 端口模式
    ————————————
     10. 启动 ShadowsocksR
     11. 停止 ShadowsocksR
     12. 重启 ShadowsocksR
     13. 查看 ShadowsocksR 日志
    ————————————
     14. 其他功能
     15. 升级脚本

    请输入数字 [1-15]：
    ```
    
- 安装过程中会询问是否设置协议和混淆插件兼容原版，是指兼容 SS，可以根据需求进行选择，原则上不推荐使用 SS 客户端，选择 n
    
    ```
    是否设置 协议插件兼容原版(_compatible)？[Y/n]n
    是否设置 混淆插件兼容原版(_compatible)？[Y/n]n
    ```
    
- 等待安装完成后，出现如下界面。这些信息是你客户端连接时需要输入的配置，以后也可以执行`bash ssr.sh`键入 5 进行查看

    ```
    ===================================================

     ShadowsocksR账号 配置信息：

     IP       : 服务器IP
     端口     : 设置的端口
     密码	    : 设置的密码
     加密	    : aes-128-ctr
     协议	    : auth_sha1_v4
     混淆	    : tls1.2_ticket_auth
     设备数限制 : 5
     单线程限速 : 0 KB/S
     端口总限速 : 0 KB/S
    ```
    
- 在客户端配置好就可以愉快的访问 Google 了！
![B19E860A-93D6-4010-BB77-6D68307B5A40](/0.png)

## 安装锐速

> 如果你想获取更加流畅的网速体验，就需要进行下一步：更换内核，安装锐速

- 安装脚本，只测试了适用于 Ubuntu 16.04，据说也适用于 Ubuntu 14.04

    ```
    wget xiaofd.github.io/ruisu.sh && bash ruisu.sh
    ```
    该脚本会更换系统内核，并自动重启，重启后不出问题的话就已经安装好了锐速并启动，可以使用`ps -ef | grep appex`查看是否在运行，同时注意 ShadowsocksR 是否已启动
    
- 安装后的文件放在`/appex`目录下，运行脚本为`/appex/bin/serverSpeeder.sh`，下面是该脚本的命令参数

    ```
    root@ubuntu:~# /appex/bin/serverSpeeder.sh 
    Usage: /appex/bin/serverSpeeder.sh {start | stop | reload | restart | status | stats | renewLic | update | uninstall}

      start		  start ServerSpeeder
      stop		  stop ServerSpeeder
      reload	  reload configuration
      restart	  restart ServerSpeeder
      status	  show ServerSpeeder running status
      stats		  show realtime connection statistics

      renewLic	  update license file
      update	  update ServerSpeeder
      uninstall	  uninstall ServerSpeeder
    ```
    需要注意的是在停止后重新运行需要`serverSpeeder.sh renewLic`生成新的 License
    
- 使用脚本查看运行状态

    ```
    root@ubuntu:~# /appex/bin/serverSpeeder.sh status
    [Running Status]
    ServerSpeeder is running!
    version              3.11.20.10

    [License Information]
    License              71B8FFD5E4F47EC5 (valid on current device)
    MaxSession           unlimited
    MaxTcpAccSession     unlimited
    MaxBandwidth(kbps)   1048576
    ExpireDate           2035-12-31
    ```
    
## 服务器测速

> 这里介绍一个服务器测速脚本：**speedtest-cli**。这是一个命令行脚本用于测试网络的上行和下行带宽。[GitHub 官方地址](https://github.com/sivel/speedtest-cli)
 
- 脚本是用 Python 写的，可以使用`pip`安装。或者直接下载运行脚本，具体看 GitHub 上的文档

    ```
    wget -O speedtest-cli https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py && chmod +x speedtest-cli
    ```

- 运行脚本，系统会自动判断你服务器所在位置，并找到最近的节点进行测速

    ```
    root@ubuntu:~# ./speedtest-cli 
    Retrieving speedtest.net configuration...
    Testing from Choopa, LLC (*.*.*.*)...
    Retrieving speedtest.net server list...
    Selecting best server based on ping...
    Hosted by Viewqwest Pte Ltd (Singapore) [6.25 km]: 1.693 ms
    Testing download speed................................................................................
    Download: 2581.24 Mbit/s
    Testing upload speed................................................................................................
    Upload: 1019.72 Mbit/s
    ```
    
## References

- [自建ss服务器教程 —— GitHub](https://github.com/Alvin9999/new-pac/wiki/%E8%87%AA%E5%BB%BAss%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%95%99%E7%A8%8B)
- [Ubuntu一键更换内核 安装锐速 —— 简书](https://www.jianshu.com/p/19ab389820ef)
- [实战vultr搭建SSR+锐速](https://www.qcgzxw.cn/1640.html/comment-page-1#comments)
- [Linux VPS 使用 speedtest 进行网络测速](https://www.linuxdaxue.com/linux-vps-net-test-by-speedtest.html)
