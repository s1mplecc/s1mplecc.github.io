---
title: 基于DPDK的模拟(D)Dos攻击实验
tags: []
categories: [Experiments]
date: 2020-09-17 21:02:12
---
## 前言

> 该实验是我暑期前往导师实验室，与一群东南大学网安三年级的本科生所做。我的工作是帮他们搭建了实验所需运行的服务器环境，实验数据的采集、制图以及结果分析均是他们所做。由于该实验可能在我研究生阶段进行进一步的研究，所以将它摘至我的博客，文档亦摘录了本科生们所做的实验报告，特此申明。

东大网安本科组员：翟思宇、肖遥、宋昌霖、胡钺琳、赵泽瑞、赵钧陶
指导老师：肖卿俊

## 引言

本实验搭载高速以太网卡，通过持续的高速数据传输，达到逼近目标服务器链路的传输速率极限，以模拟 (D)Dos 攻击的效果。攻击主机系统采用 Ubuntu Server 版本，受 Linux 系统 I/O 的内核机制的影响，发包速率受到限制。本实验创新性的利用 DPDK 软件平台绕过系统内核，实现高数据吞吐率的效果。

![81db04d3b1658e9c254bfbfce6dedde2.png](0.png)

如上图，Linux 内核协议栈的实现决定了它内核网络协议的性能不佳，Linux 在内核收包处理时，将网卡收到的报文通过 DMA 放到内存，网卡出发中断通知系统有报文到达，系统分配 sk_buff，将报文拷贝到 sk_buff 中，交由协议栈处理，之后将其送用户态应用程序处理。

这种情况下，报文数量的增多将急剧增加资源的消耗。这包含报文产生 CPU 中断的上下文切换、为报文申请分配 sk_buff 消耗的资源、用户态程序收发包时产生系统调用和上下文切换带来的系统资源消耗。Kernel Bypass（内核旁路）技术应运而生，DPDK 正是采用的上图图由这种类 DMA 机制，直接将数据包从硬件（网卡端口）传输至用户态应用程序，以此来实现低延迟、低消耗的高吞吐网络传输。

TRex 是思科研发的一款成熟的基于 DPDK 的网络测试工具。它运行在标准的 Intel 处理芯片上，同时支持 stateful 和 stateless 两种模式，stateful 可以描述 L4~L7 层的应用场景，而 stateless 主要用于进行定制包的发包。本实验主要利用 TRex 的 DPDK 高线速发包能力，模拟对目标服务器进行 (D)Dos 打击。

![7a135e990f639b406cd1cc244d4943f3.png](1.png)

## 实验物理环境

![2ffa174d247d24ce9f16ea7332178308.jpeg](2.JPG)

1. DELL Xeon Server，OS: Ubuntu Server 18.04 LTS。原装的系统为 CentOS 6，实验过程中遇到内核版本过低不支持 TRex 发包的问题，所以重装了 Ubuntu 系统。此外服务器还安装了实验室购入的兼容 DPDK 的 INTEL X710 网卡。
2. NetFPGA-SUME Virtex-7 FPGA Development Board，四个 10Gbps 的端口，实验过程中使用两根光缆连接到服务器构成回环。
3. DELL 工作站，用于将编译好的 P4 程序烧入 NetFPGA 智能网卡。

## 服务器环境准备

### 安装系统

1. 服务器操作系统选择兼容性、稳定性较好的 Ubuntu 18.04.5 LTS 版本，官网下载需要翻墙，也可以选择清华或阿里的开源软件镜像网。[官网下载地址](https://releases.ubuntu.com/bionic/)
2. 制作启动 U 盘，使用 Rufus 镜像刻录工具将下载好的 iso 镜像刻录至 U 盘。[官网教程 - Create a bootable USB stick on Windows](https://ubuntu.com/tutorials/create-a-usb-stick-on-windows#1-overview)。
3. 启动服务器，按住 F2 进入 DELL 的 BIOS 界面，将 U 盘调整为启动的第一选项。
4. 重启服务器，按照引导程序安装系统。[官网教程 - Install Ubuntu Server](https://ubuntu.com/tutorials/install-ubuntu-server#1-overview)

### 连接外网

![8d45ed82634b33c68b4ddacfeef8a6a7.jpeg](3.JPG)

服务器共有四个网口 eno1～eno4，将网线插在左数第一个网口对应 eno1（图中①），并配置 interfaces 文件。图中②位置为 INTEL X710 网卡位置，从 NetFPGA 智能网卡的端口连接至服务器。

```
$ vi etc/network/interfaces

auto eno1
iface eno1 inet dhcp
```

在`/etc/resolv.conf`中加入 DNS 配置，此为阿里提供的公共 DNS

```
nameserver 223.5.5.5
nameserver 223.6.6.6
```

要先手动设置 IP 再使用 `dhclient` 命令启用 DHCP，**每次重启服务器后如需连接外网，需输入这两行命令**

```
ifconfig eth0 192.168.0.102 netmask 255.255.255.0
dhclient eno1
```

执行完成就可以连通外网了

### 设置网口

`ifconfig -a `查看网口信息，列出了 X710 网卡的两个网口信息，默认驱动是 Kernel Driver i40e，先设置 IP

```
sudo ifconfig enp59s0f0 192.168.200.2
sudo ifconfig enp59s0f1 192.168.201.2
```

在工作站的主机上 ping 一下服务器保证双向联通

### 安装依赖

使用 `apt-get` 命令安装 DPDK 依赖

```
sudo apt update
sudo apt-get upgrade
sudo apt install -y dpdk dpdk-dev dpdk-doc
```

安装其他依赖，可以先在本地保存为 `install.sh`，然后使用 `scp` 命令传到服务器上 `sh install.sh`执行

```
sudo apt-get install -y cmake gcc g++ git automake llvm llvm-dev llvm-runtime libtool bison flex build-essential vim

# Install pkg-config here, as it is required for p4lang/PI
# installation to succeed.
sudo apt-get install -y pkg-config

sudo apt-get install -y wget curl zip unzip rar unrar unar

sudo apt-get install -y libgc-dev libfl-dev libgmp-dev libevent-dev libssl-dev libjudy-dev libpcap-dev tcpdump

sudo apt-get install -y libboost-dev libboost-iostreams-dev libboost-graph-dev libboost-test-dev libboost-program-options-dev libboost-system-dev libboost-filesystem-dev libboost-thread-dev

sudo apt-get install -y libreadline6 libreadline6-dev
# 这里如果提示废弃就执行下面的安装
sudo apt-get install -y libreadline-dev 

# Deps needed to build PI:
sudo apt-get install -y libjudy-dev libreadline-dev valgrind libtool-bin libboost-dev libboost-system-dev libboost-thread-dev

# Things needed for `cd tutorials/exercises/basic ; make run` to work:
sudo apt-get install -y libgflags-dev net-tools

sudo apt-get install -y doxygen graphviz texlive-full

sudo apt-get install -y bridge-utils tcpreplay
sudo apt-get install -y zlib1g-dev pciutils kmod strace  ## needed by cisco trex
```

### 安装TRex

联网下载 TRex

```
wget --no-cache http://trex-tgn.cisco.com/trex/release/latest
tar -zxvf latest
```

我们下载的 TRex 最新版本为 v2.82，进入到该目录下并执行 DPDK 端口设置脚本

```
cd v2.82
./dpdk_setup_ports.py -i
```

按照下图设置

![0b9649d536fbab09908a9df0303c590a.jpeg](4.jpeg)

此时可以注意到，X710 网卡的两个网口还是绑定的 i40e 内核驱动。但在运行 TRex 发包命令后会自动绑定到 DPDK 兼容驱动 igb_uio 上。

## 发包测试

实验环境准备就绪，进行发包测试。参考 TRex 官方手册 [Running examples
](https://trex-tgn.cisco.com/trex/doc/trex_manual.html#_running_examples)，在 stateful 模式下使用 `t-rex-64` 命令发送数据包。`-f` 指定配置 yaml，`-m` 指定重放次数，`-l` 指定网络延迟检测速率。更多参数详见官方手册。

```
sudo ./t-rex-64 -f avl/sfr_delay_10_1g.yaml -m 5 -l 1000
```

测试效果如下：

![7402f2999111d19e378f8883aaf3ba34.jpeg](5.jpeg)

```
-Per port stats table 

      ports |               0 |               1 
 -----------------------------------------------------------------------------------------

   opackets |        27882688 |        35434728 
     obytes |      8217921859 |     28586813888 
   ipackets |         5007937 |             709 
     ibytes |      4647534484 |           79608 
    ierrors |               0 |               0 
    oerrors |               0 |               0 
      Tx Bw |       1.18 Gbps |       3.81 Gbps 

-Global stats enabled 
 Cpu Utilization : 31.6  %  31.6 Gb/core 
 Platform_factor : 1.0  
 Total-Tx        :       5.00 Gbps  
 Total-Rx        :       0.00  bps  
 Total-PPS       :       1.07 Mpps  
 Total-CPS       :      20.51 Kcps  

 Expected-PPS    :       1.08 Mpps  
 Expected-CPS    :      20.61 Kcps  
 Expected-BPS    :       5.02 Gbps  

 Active-flows    :    21328  Clients :      511   Socket-util : 0.0775 %    
 Open-flows      :  1342791  Servers :     5621   Socket :    24927 Socket/Clients :  48.8 
 drop-rate       :       5.00 Gbps   
 current time    : 66.5 sec  
 test duration   : 3533.5 sec  

-Latency stats enabled 
 Cpu Utilization : 0.1 %  
 if|   tx_ok , rx_ok  , rx check ,error,       latency (usec) ,    Jitter          max window 

   |         ,        ,          ,     ,   average   ,   max  ,    (usec)                     
 ---------------------------------------------------------------------------------------------------------------- 

 0 |    65254,   12858,         0,    0,          7  ,      23,       1      |  10  10  15  10  12  11  12  23  10  10  15  18  12 
 1 |    65254,     136,         0,    0,          3  ,       0,       0      |  0  0  0  0  0  0  0  0  0  0  0  0  0 

## 
```

主要记录的数据包括：

- **Cpu Utilization**，CPU利用率
- **Total-Tx**，总计发送速率（Transport）
- **Total-Rx**，总计接受速率（Receive）
- **Total-PPS**，总计每秒传输包数量（Packets per second）

## 发包实验

接下来我们尝试在 stateful 模式下模拟 stateless 发包，使用的是 `cap2/` 目录下的 `imix.yaml` 配置文件，根据以太网协议，一个数据包的大小从最小 64Bytes 到最大 1518Bytes，imix 提供了 imix_64、imix_594 和 imix_1518 三类配置文件。

`-c` 指定 CPU 核数，观察并绘制不同包大小、不同核数时 CPU 利用率以及吞吐率的 eps 矢量图。

```
sudo ./t-rex-64 -f cap2/imix_1518.yaml -m 823451 -l 1000 -c 2
```

实验结果如下：

```
-Per port stats table 

      ports |               0 |               1 
 -----------------------------------------------------------------------------------------

   opackets |        34232892 |           42080 
     obytes |     51904428378 |         2777280 
   ipackets |               0 |               0 
     ibytes |               0 |               0 
    ierrors |               0 |               0 
    oerrors |               0 |               0 
      Tx Bw |       9.89 Gbps |     527.55 Kbps 

-Global stats enabled 
 Cpu Utilization : 100.0  %  9.9 Gb/core 
 Platform_factor : 1.0  
 Total-Tx        :       9.89 Gbps  
 Total-Rx        :       0.00  bps  
 Total-PPS       :     816.02 Kpps  
 Total-CPS       :       0.00  cps  

 Expected-PPS    :       6.59 Gpps  
 Expected-CPS    :       6.59 Gcps  
 Expected-BPS    :      80.00 Tbps  

 Active-flows    :     1600  Clients :      254   Socket-util : 0.0100 %    
 Open-flows      :     1600  Servers :    65534   Socket :     1600 Socket/Clients :  6.3 
 Total_queue_full : 72264932         
 drop-rate       :       9.89 Gbps   
 current time    : 43.3 sec  
 test duration   : 3556.7 sec  

-Latency stats enabled 
 Cpu Utilization : 0.1 %  
 if|   tx_ok , rx_ok  , rx check ,error,       latency (usec) ,    Jitter          max window 

   |         ,        ,          ,     ,   average   ,   max  ,    (usec)                     
 ---------------------------------------------------------------------------------------------------------------- 

 0 |    42080,       0,         0,    0,          0  ,       0,       0      |  0  0  0  0  0  0  0  0  0  0  0  0  0 
 1 |    42081,       0,         0,    0,          0  ,       0,       0      |  0  0  0  0  0  0  0  0  0  0  0  0  0 

## 
```

## 实验结果

### 理论分析

一个以太网帧的结构包括：

| PA | SFD | DA | SA | Type | Payload | FCS | IFG |
|----|-----|----|----|------|---------|-----|-----|
| 7  | 1   | 6  | 6  | 2    | 46 ~ 1500 | 4   | 12  |

最小帧长：7+1+(6+6+2+46+4)+12 = 84B（最小有效帧长 64B）
最大帧长：7+1+(6+6+2+1500+4)+12 = 1538B（最大有效帧长 1518B）

对于最小帧长，最大包传输率 M 和吞吐率 T 为：

$$M = Speed/Size = 100 * 10^9 / (84 * 8) = 148809523 pps$$
$$T = M * 64 * 8 = 76.19Gbps$$

对于最大帧长，最大包传输率 M 和吞吐率 T 为：

$$M = Speed/Size = 100 * 10^9 / (1538 * 8) = 8127438 pps$$
$$T = M * 1518 * 8 = 98.69Gbps$$

以此类推，我们对于不同的帧长做了理论分析，得到下图：

![38021550bd20f375c0c872a80cea41af.png](6.png)

### 吞吐率测试结果

以下是吞吐率的测试结果：

![037b37c99b1bc4c938f887dc1b876ba1.png](7.png)
![ec784c4bc84e984d27170739145d5bfe.png](8.png)
![d9c6568a7c11555a9290d452fbacafe2.png](9.png)

### 包传输率测试结果

以下是包传输率的测试结果：

![0f5b53c8b5544ae9572cebc9168f12cb.png](10.png)
![83c9995318589576075bf3140b4f46fd.png](11.png)
![79d66dcca878a695f31c01cf8e08e2d8.png](12.png)

### 与内核机制发包的对比

| 包大小\(B\) | 内核机制发包<br>CPU利用率 | 内核机制发包<br/>PPS\(Kpps\) | DPDK机制发包<br/>PPS\(Mpps\) |
| :---------: | :-----------------------: | :--------------------------: | :--------------------------: |
|     64      |           100%            |            593\.5            |            14\.85            |
|     549     |           100%            |           585\.29            |            2\.04             |
|    1518     |          97\.70%          |           537\.27            |            0\.82             |

## 结论

由上述实验结果可知：
1. 10Gbps 线速的网络，用 DPDK 跑 4 个核，基本可以实现 64B 最小包较好的发送吞吐率，但是仍达不到 10Gbps 的线速，只有 7.6Gbps 左右。
2. DPDK 由于采用了绕过内核驱动的技术，使得网络数据包的发送速度大大增加。
3. 此外，随着数据包长度增加，其他条件一定时，吞吐率也随之增加，这与我们的理论分析基本吻合。

## 参考

- *Surasak Sanguanpong, Experiences in Building a 100 Gbps (D)DoS Traffic Generator, DIY with a Single  Commodity-off-the-shelf (COTS) Server.*
- [TRex official manual](https://trex-tgn.cisco.com/trex/doc/trex_manual.html)
- [TRex Stateless Support](https://trex-tgn.cisco.com/trex/doc/trex_stateless.html#_stateful_vs_stateless)
- [思科TRex Traffic Generator代码仓库](https://github.com/cisco-system-traffic-generator/trex-core)