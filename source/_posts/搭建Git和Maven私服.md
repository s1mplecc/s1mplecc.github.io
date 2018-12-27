---
title: 搭建 Git 和 Maven 私服
date: 2017-11-21T04:52:17.000Z
tags: ['Server', 'Git', 'Maven']
categories: [Ops]
---
## Preface

> 应公司需求，搭建Git和Maven的企业私服，Maven私服使用Nexus搭建。

## Prerequisites

- 一台服务器，4核8G、1TB硬盘，Ubuntu 16.04
- apache-maven-3.5.2-bin.tar.gz
- nexus-3.6.1-02-unix.tar.gz，[官网下载](https://www.sonatype.com/download-oss-sonatype)
- jdk-8u151-linux-x64.tar.gz

## 搭建Git

- 安装Git，`git --version`查看版本
    ```
    sudo apt-get install git
    ```

- 创建git用户，该命令会创建git组、git用户，并设置`/home/git`为用户目录
    ```
    adduser git
    ```
    
- 为了安全，禁止git用户bash登录，`vi /etc/passwd`
    ```
    git:x:1001:1001:,,,:/home/git:/bin/bash
    ```
    修改为
    ```
    git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
    ```

#### Tips

> **git-shell :** 提供了受限的git访问，只允许**push/pull**命令，官网解释：
> This is a login shell for SSH accounts to provide restricted Git access. It permits execution only of server-side Git commands implementing the pull/push functionality, plus custom commands present in a subdirectory named git-shell-commands in the user’s home directory.

## 测试Git

- 在`/opt`创建文件夹，`mkdir git-repo`，并`cd`进去

- 初始化一个Git仓库，在当前目录下生成了一个空的Git仓库，包含了一些初始文件
    ```
    git init --bare helloworld.git
    ```
    
- 更改权限
    ```
    chown -R git:git helloworld.git 
    ```
    
- 将用户的ssh公钥加入到`/home/git/.ssh/authorized_keys`中，并重启sshd服务
    
- 在**本地机器**上将其**clone**下来
    ```
    git clone git@{IP}:/opt/git-repo/helloworld.git
    ```
    会发现当前目录下多了个名为helloworld的文件夹
    
- 现在我们添加一个文件，测试将其push到服务器上
    ```
    touch abc.txt
    git add .
    git commit -m "first commit"
    git push -u origin master
    ```
    
## 搭建Maven私服

> Nexus是目前Sonatype公司开发的搭建Maven企业级私服的工具，有付费版本**Pro**和开源版本**OSS** (Open Source Software)

- 上传jdk、maven和nexus的tar包并解压改名，如下
    ```
    root@git-maven:/opt# ls
    git-repo  jdk  maven  nexus sonatype-work
    ```
    其中`sonatype-work`文件夹用于存放私服的配置，日志文件和依赖包

- 修改环境变量，`vi /etc/profile`
    ```
    # maven
    export M3_HOME=/opt/maven
    export PATH=$M3_HOME/bin:$PATH

    # jdk
    export JAVA_HOME=/opt/jdk
    export JRE_HOME=${JAVA_HOME}/jre
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
    export PATH=${JAVA_HOME}/bin:$PATH

    # nexus
    export PATH=/opt/nexus/bin:$PATH
    ```
    
- 使环境变量生效，`source /etc/profile`，现在可以执行`nexus`命令查看它的一些操作，如下
    ```
    Usage: nexus { console | start | stop | restart | status | dump }
    ```
    
- nexus不建议使用root用户启动，所以我们新建nexus用户，`adduser nexus`。**注意**，关于nexus的操作最好都在nexus用户下执行

- 切换到nexus用户，`su - nexus`，启动nexus
    ```
    nexus start
    ```
    
- 如果已正常启动，访问`{IP}:8081`即可前往控制台，8081为nexus默认端口，默认用户名admin，密码admin123，可在控制台中修改密码
![-----2017-11-22---4.40.57](/0.png)

#### Tips

> 其中**maven-public**是一个组，包含了**maven-central**(代理中央仓库，存放依赖构件)、**maven-releases**(存放发行版本)、**maven-snapshots**(存放快照版本)。


## 导入依赖包

- [Maven中央仓库网址](http://mvnrepository.com/)

> 现在已经成功部署了Nexus。可以通过配置你的项目，使其从你的私服上拉取依赖包，如果拉取不到，它会到Maven中央仓库拉取依赖并保存在服务器上

- 新建Maven项目，修改`pom.xml`
    ```
    <repositories>
        <repository>
            <id>nexus</id>
            <name>nexus</name>
            <url>http://{IP}:8081/repository/maven-public/</url>
        </repository>
    </repositories>
    ```
    
- 在`<dependencies>`中添加你需要的依赖，执行`mvn package`打包项目即可

- 控制台上查看服务器上已经存储的依赖包
![-----2017-11-22---4.41.17](/1.png)

## 搭建GitLab

- [清华大学开源软件镜像站](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/)

> 有空搭了个Gitlab，可以方便管理项目和成员公钥。但是运行GitLab对服务器内存的要求较高。官网明确要求内存至少4G
    
- 信任GitLab的GPG公钥
    ```
    curl https://packages.gitlab.com/gpg.key 2> /dev/null | sudo apt-key add - &>/dev/null
    ```
    
- 添加清华的镜像源，`touch /etc/apt/sources.list.d/gitlab-ce.list`
    ```
    deb https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu xenial main
    ```
    
- 安装gitlab-ce
    ```
    sudo apt-get update
    sudo apt-get install gitlab-ce
    ```

- 修改GitLab的Url，`vi /etc/gitlab/gitlab.rb`
    ```
    external_url 'http://{domain or IP}'
    ```

- 启动GitLab
    ```
    sudo gitlab-ctl reconfigure
    ```
    
> 访问你的Url进行注册登录即可，可以创建Groups、Projects，并添加你的SSH公钥。另外修改root用户密码参考文章：http://blog.csdn.net/yin138/article/details/51394868

