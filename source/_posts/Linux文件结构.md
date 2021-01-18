---
title: Linux 文件结构
date: 2018-01-30T02:28:54.000Z
tags: ['Linux']
categories: [Ops]
---
## 前言

初学 **Linux** 总是好奇 **Linux** 系统下各个文件夹为什么这么命名，它们的用途是什么。本文主要介绍 **Linux** 下各文件夹的结构和用途说明

使用的环境为 **Aliyun** 的服务器，系统为 **Ubuntu 16.04** 版本

不同的系统文件结构必然有所不同，但是只要是 **Linux** 发行版，就应该遵循 **Linux** 文件系统

## Color

一般人性化点的 **Shell** 都会有文件的不同颜色显示。下图为 **Linux** 下不同颜色的文件代表的含义。比如可执行文件显示绿色，目录显示蓝色，链接文件显示浅蓝色等
![linuxfilecolor](/0.png)

## FHS

**Linux** 有专门的指导文件系统的标准，叫做 **FHS**(or **Filesystem Hierarchy Standard**)

**Linux** 所依赖的文件系统，有两个基本的概念：

```
shareable / unshareable （共享性）
static / variable （稳定性）
```

对于可分享（**shareable**）的文件系统，**FHS** 规定
可分享文件能够被存放在一台将被多个用户同时访问的主机上，通常情况下，当一个系统所需的所有文件都存放在一台外部主机上时，可以方便的挂载一个或很少几个目录来使这些文件可用。但是，并不是在文件系统架构中的所有文件都是可分享的，所以**每个系统都至少需要一处能够存放不可共享文件**的本地空间

对于静态（**static**）文件系统，**FHS** 规定
**静态文件需要与动态文件隔离**，因为，不同于动态文件，静态文件可以存放在**只读介质**上（而动态文件不可以），并且不需要和动态文件一起备份

**FHS** 中关于特定文件夹用途和所需包含内容的规定

```
/bin: Essential user command binaries (for use by all users)
/sbin: System binaries
/usr: shareable static
    /usr/bin: Most user commands
    /usr/sbin: Non-essential standard system binaries
    /usr/local: Local hierarchy
```

### /bin

`/bin`，**binary** 的简称。该目录下为**二进制**的**可执行文件**，显示为**绿色**。包含了引导启动所需的命令或普通用户可使用的命令。多是系统中重要的**系统文件**

比如说根目录`/bin`下都是些常用命令的执行文件，再比如说 **Tomcat** 的`bin`目录下有一些运行脚本，有 **Linux Shell** 执行的`.sh`结尾的文件，也有 **Windows** 执行的`.bat`结尾的文件

```
$ ls /bin/
cat    tar    cp      ls     rm   ...

$ ls /usr/local/tomcat9/bin/
shutdown.bat    catalina.bat     shutdown.sh
catalina.sh     startup.bat      startup.sh    ...
```

### /sbin

`/sbin`，**System binaries** 的简称。存放**系统管理员**以及其他需要**root权限**来运行的工具

```
$ ls /sbin/
reboot    shutdown    poweroff    route    sysctl     ...  
```

### /usr

`/usr`是文件系统中第二大主要部分，其中存放了**可分享的**、**只读的**数据（即**可分享性**，**静态性数据**）。这说明`/usr`文件系统需要能够在各种不同的兼容 **FHS** 系统的之间可共享，并且不可被修改。任何依赖于平台或者需要改写的数据，都应该存放在其它位置

`/usr/bin`集中了几乎所有用户命令，比如`man、info`系统自带或者`git、docker`这种后来安装的三方软件的命令

`/usr/sbin`包括了多数服务程序需要管理员权限的命令，比如`mysqld、sshd`

`/usr/local`，本地安装的软件和其他文件都可以放在这里

### /lib

`/lib`，**Libraries** 的简称。库文件，存储文件运行需要的依赖库。不如 **Tomcat** 是基于 **Java** 的，它的`lib`目录下包含了一些依赖 **Jar** 包

```
$ ls /usr/local/tomcat9/lib
annotations-api.jar       el-api.jar       tomcat-coyote.jar   tomcat-util-scan.jar
catalina-ant.jar          jasper-el.jar 
```

### /etc

`/etc`这个文件夹很有意思，也是最开始困扰我很久的，因为它里头什么都有...其实它取名就是取自"**等等**"的缩写词 **etc.** 存放着系统以及各种程序的配置文件

```
$ ls -F /etc
hosts    hostname    passwd    nginx/    ssh/    ...
```

早期 **UNIX** 系统中，对于`/etc`，贝尔实验室的解释是：**etcetra directory**(or **etc.**) 。表示其他、等等什么的，其实就是用来放不能归类到其他目录中的内容，所以比较杂乱。后来 **FHS** 规定用来放配置文件，就解释为：**Editable Text Configuration** 或者 **Extended Tool Chest**

### /var

`/var`，**Variable** 的简称，多用于存放动态变化的文件。也用于某些大文件的**溢出区**，比如各种服务的日志文件

```
$ ls /var/backups
group.bak    passwd.bak    ....

$ ls /var/log
syslog    dpkg.log    auth.log     ...

$ ls -F /var/run
nginx.pid    docker.pid    sshd.pid     resolvconf/    ...
```

`/var/run`中包含了进程的ID以及动态生成的配置文件等，比如连接网络时自动生成的`/var/run/resolvconf/resolv.conf`域名解析服务器的配置信息，其实`/etc/resolv.conf`文件就是链接到此文件

### /root

`/root`目录是`root`用户的**用户目录**，`~`代表当前用户的用户目录，普通用户的目录在`/home`下，比如我通过`su s1mple`切换到 s1mple 用户，用户目录为`/home/s1mple`

```
root@ubuntu-server:~# pwd
/root

s1mple@ubuntu-server:~$ pwd
/home/s1mple
```

### 其他

`/opt`，**Optional application software packages** 代表你可以在该目录下安装第三方软件，和`/usr/local`的作用相似

`/dev`，**device** 的简称，该目录存放设备文件，即设备驱动程序，用户通过这些文件访问外部设备。比如，用户可 以通过访问`/dev/mouse`来访问鼠标的输入，就像访问其他文件一样

`/tmp`，**temporary** 的简称，目录存放程序在运行时产生的临时数据，**注意：**`/var/tmp`比`/tmp`允许更大的或需要存在较长时间的临时文件

`/proc`，虚拟的目录，是系统**内存**的映射。可直接访问这个目录来获取系统信息，比如`/proc/cpuinfo`

## 参考

- [Linux 下各文件夹的结构说明及用途介绍](http://mp.weixin.qq.com/s/yUwUawPlIjPThe3v1Ppcug)
