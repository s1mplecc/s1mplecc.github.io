---
title: 安装 JDK&Tomcat
date: 2017-11-02T14:17:06.000Z
tags: ['Linux', 'Java']
categories: []
---
## Preface

> Tomcat是现在主流的应用服务器。可以直接将基于Spring框架的Java Web应用部署到Tomcat上。本文将介绍如何安装JDK和Tomcat

## Prerequisite 

- 服务器：Ubuntu 16.04
- 良好的网络环境

## Install JDK

> 官网安装jdk，相对较麻烦，但是官网毕竟权威

- [oracle官网](http://www.oracle.com/technetwork/java/javase/downloads/index.html)复制jdk下载链接，执行`wget`命令即可

- 解压`.tar.gz`压缩包，并改名
    ```
    mv jdk-8u151-linux-x64.tar.gz /usr/local/lib/
    tar -xzvf jdk-8u151-linux-x64.tar.gz
    mv jdk-8u151-linux-x64 jdk8
    ```
    
- `vim ~/.bashrc`配置环境变量，[为什么要设置Java环境变量](http://www.cnblogs.com/wangchenyang/archive/2011/08/17/2143620.html)
    ```
    export JAVA_HOME=/usr/local/lib/jdk8
    export JRE_HOME=${JAVA_HOME}/jre
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
    export PATH=${JAVA_HOME}/bin:$PATH 
    ```
    
> **bashrc**与**profile**都用于保存用户的环境信息，**/etc/profile** 中设定的变量(全局)的可以作用于任何用户。**~/.bashrc** 该文件包含专用于某个用户的bash shell的bash信息,当该用户登录时以及每次打开新的shell时,该文件被读取。需要注意的是我在docker容器中的 **/etc/profile** 配置java环境变量时，重启容器发现配置没生效，应该是操作系统在重启时加载 **/etc/profile** 文件，但是docker容器就没有这种操作了。
    
- `source ~/.bashrc`使文件生效

#### **Tips**

1. 当你执行一条命令，系统会去PATH路径下查找该命令，比如你现在可以执行`java -version`命令。`echo $PATH`查看PATH设置。   
2. 全局环境变量，可以`${}`或者`$`引用，你现在可以`echo ${JAVA_HOME}`打印该值。`env`查看环境变量设置。
    
> 如果你觉得官网太麻烦，可以安装openjdk，虽说是阉割版，但足够满足你的需求了

- 安装**openjdk**只需要下面一行命令
    ```
    apt-get install openjdk-8-jdk
    ```
    
#### **Tips**
    
1. 你可以通过如下命令查看有关jdk版本和信息。
    ```
    apt-cache search jdk
    apt-cache show openjdk-8-jdk
    ```
2. java有关的可执行文件被安装在`/usr/bin`目录下，但是环境变量并没有设置。

## Install Tomcat

- 照例，[官网](https://tomcat.apache.org/download-90.cgi)下载、解压。这里不再赘述

- 配置Tomcat为服务，`cd /usr/local/lib/tomcat9`目录下
    ```
    cp ./bin/catalina.sh /etc/init.d/tomcat
    ```
- `vim /etc/init.d/tomcat`加入如下内容
    ```
    JAVA_HOME=/usr/local/lib/jdk8
    JRE_HOME=${JAVA_HOME}/jre
    CATALINA_HOME=/usr/local/lib/tomcat9
    CLASSPATH=.:${JAVA_HOME}/lib:${CATALINA_HOME}/lib
    ```
- 添加**restart**命令，`"$1"`指输入的第一个参数
    ```
    elif [ "$1" = "restart" ] ; then
        service tomcat stop
        sleep 1
        service tomcat start
    ```    
- 保存退出，现在可以通过`service tomcat [start|stop|restart]`来启动、停止Tomcat。查看是否正常启动
    ```
    curl localhost:8080
    ```

#### **Tips**

1. 你的应用程序安装路径在**webapps**下。在部署你自己的应用前执行`rm -r webapps/*`清空文件夹    
2. Tomcat的配置文件在`conf/server.xml`中

### **About server.xml**

- [server.xml详解](http://www.cnblogs.com/gugnv/archive/2012/02/01/2334187.html)

- 默认开启8080端口，可以修改端口号
    ```
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    ```

- `<Host>`标签中，**appBase**解释了为什么将Java Web程序打包放在此目录下。**unpackWARs**为true自动解压war包，**autoDeploy**自动部署。所以在Tomcat启动后识别webapps目录下`.war`结尾的war包自动解压、部署。
    ```
    <Host name="localhost"  appBase="webapps"
          unpackWARs="true" autoDeploy="true">
    ```

- 在`<Host>`标签中可设置`<Context>`上下文标签。**docBase**是应用程序包名，**path**是url前缀，**reloadable**为true时自动检测应用程序的`/WEB-INF/lib`和`/WEB-INF/classes`目录的变化，自动装载新的应用程序
    ```
    <Context docBase="app" path="/app" reloadable="true" />
    ```
    
- 如果不配置`<Context>`标签，将默认以应用程序文件夹名字作为url前缀，如**webapps**目录下应用程序文件夹名为**jpress**，则url为
    ```
    http://{IP}:{port}/jpress/
    ```



