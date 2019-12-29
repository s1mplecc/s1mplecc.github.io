---
title: 搭建 SSR 科学上网
tags: []
categories: [Ops]
date: 2019-04-10 15:02:32
---
## 声明

本教程也是由于本人平时工作学习有 Google 的需求，所以就自己搭了一个梯子，并记录了搭建的过程方便以后重搭。大家搭建后小规模使用就好了，这样网速较快且不容易被封。**特此申明：严禁用于商业用途！**

## 准备

- **VPS 服务器**，需要选择国外的，推荐 [Vultr](https://www.vultr.com/)，速度不错、稳定且性价比高，按小时计费，能够随时开通和删除服务器。**操作系统选择 CentOS 6**，为了更换内核方便不宜选择版本过高的操作系统。
- MacOS 自带的 Terminal，或者 Windows 的 XShell。用于 ssh 连接远程服务器并执行命令。
- **ShadowsocksR 客户端**，简称 SSR，是 Shadowsocks 的增强版，在其基础上增加了一些数据混淆方式。SSR 下载地址：[Mac 版](https://github.com/shadowsocksr-backup/ShadowsocksX-NG/releases)，[Windows 版](https://github.com/shadowsocksr-backup/shadowsocksr-csharp/releases)。

## 部署SSR服务端

执行 `ssh root@{IP}` 连接上 VPS 服务器，并安装 SSR 服务端一件部署脚本

```sh
yum -y install wget

wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssr.sh && chmod +x ssr.sh && bash ssr.sh
```

等待下载完成后出现如下界面，按照提示一步步进行配置

![6e65c95d5ad590eec8be684361594373.png](0.png)

依次会配置：

1. 端口和密码（端口建议设置 1000 ～ 65535 之间）
2. 加密方式（随意）
3. 协议插件（随意）
4. 是否兼容原版 SS 客户端（即兼容没有协议和混淆的客户端，建议选 N）
5. 每个端口的连接设备数限制、速度限制（auth_* 系列协议，并且不兼容原版时才有效）

配置完成后等待部署完成，终端会打印配置信息，这些就是需要在 SSR 客户端填入的配置。现在可以直接进入下一个章节：配置SSR客户端。

![6a90ecf445798cb2c69a8bc05abd0a0a.png](1.jpeg)

### 修改配置

默认情况下部署的是单端口模式，即多人共享一个端口。可以切换为多端口模式，一个用户一个端口，方便管理。直接执行 `bash ssr.sh` 键入 9 即可。

此外，键入 7 你可以随时修改 SSR 服务端的配置，以及在多端口模式下添加新用户，为他们分配新的端口号和密码。

![17f12e5ebc74a4105bacbeb69662a1f0.png](2.jpeg)

## 配置SSR客户端

打开 SSR 客户端，按照服务端的配置填入对应配置。

![7a34639c35446ec3808356b7eef07dc4.png](3.png)

至此就可以 Google 了，但网速较慢，可以使用锐速进行加速。锐速也有一键部署脚本，加速后对速度的提升很明显，所以推荐部署加速脚本。该加速方法是开机自动启动，部署一次就可以了。

## 锐速加速

### 更换操作系统内核

安装锐速前需要先更换操作系统内核，下载更换内核脚本并执行

```sh
wget --no-check-certificate https://blog.asuhu.com/sh/ruisu.sh && bash ruisu.sh
```

成功替换内核后服务器会自动重启。等待重启完成后，重连服务器。

### 一键安装锐速

```sh
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/serverspeeder/master/serverspeeder-all.sh && bash serverspeeder-all.sh
```

等待安装完成后即可体验飞一般的上网速度。

## 备份

Vultr 提供方便快捷的快照 Snapshot 功能用于备份服务器。为防止服务器 IP 被封，我们可以备份服务器，方便将来重新部署。

在 Vultr 用户界面的 Servers 中添加快照，快照创建需要一点时间

![1d8690c7a6d4d85711e11f2485996ba8.png](4.png)

将来在 Deploy new instance 新建实例时，服务器类型选择快照即可

![1c652a6a3177df0e85958f81faa58043.png](5.png)

## 参考

- [自建ss服务器教程](https://github.com/Alvin9999/new-pac/wiki/%E8%87%AA%E5%BB%BAss%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%95%99%E7%A8%8B)
