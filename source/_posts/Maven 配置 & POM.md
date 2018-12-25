---
title: Maven 配置 & POM
date: 2017-12-05T02:00:52.000Z
tags: ['Maven']
categories: []
---
> 关于**Maven**的**settings.xml**以及**pom.xml**的介绍

# settings.xml

- **settings.xml**会在两个位置
    `${MAVEN_HOME}/conf/settings.xml`全局配置，该文件里有很全面的英文注释
    `${USER_HOME}/.m2/settings.xml`个人配置，建议将全局文件拷贝过来再修改。加载时会覆盖全局设置

- 修改镜像源为aliyun，这样Maven会从aliyun拉取依赖，而不是速度巨慢的官方中央仓库，**id**和**name**都是自己取的用作唯一标识
    
    ```
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    </mirror>
    ```

    这样的效果和在项目中的**pom.xml**中加入如下内容的作用是一样的，只不过是针对全局所有项目的
    ```
    <repositories>
        <repository>
            <id>nexus-aliyun</id>
            <name>Nexus aliyun</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        </repository>
    </repositories>
    ```

- 如果你搭建了Maven私服，你可能会在**pom.xml**中添加 `<repositoriy>`去下载和部署Jar包，需要进行认证的**username**和**password**不应该放在pom文件中而是放在**settings.xml**的`<server>`标签中

    ```
    <server>
      <id>xxx</id>
      <username>xxx</username>
      <password>xxx</password>
    </server>
    ```
    
#### Other Settings

- `<localRepository>`定义了Maven的本地仓库地址，默认值`${USER_HOME}/.m2/repository`
- `<interactiveMode>`定义Maven与用户输入的交互模式，默认为true
- 更多参考[官方文档](http://maven.apache.org/settings.html)

---

# pom.xml

> POM全称**Project Object Model**， 它是一个名为**pom.xml**的Maven项目的XML表示形式。Maven用它来管理项目的配置、依赖等等。[官方文档](http://maven.apache.org/pom.html)

### Maven Coordinates

- Maven项目需要`groupId:artifactId:version`定义项目的坐标（Coordinates），除非继承自一个**parent**时可以省略**groupId**和**version**

- 可以看到在执行`mvn install`后，项目会被打包安装到本地Maven仓库中去，比如`/Users/s1mple/.m2/repository/com/caacetc/spark-dao/1.0-SNAPSHOT/spark-dao-1.0-SNAPSHOT.jar`，其中`com.caacetc`是我的**groupId**、`spark-dao`是我的**artifactId**、`1.0-SNAPSHOT`是版本号

- `<packaging>`打包的方式，默认为jar、还可以打包war和pom等。打包成pom不会生成目标文件，通常用于**parent**模块中。

### Dependencies

- Maven最强力的地方之一就是对于依赖的管理，Maven会自动下载依赖的Jar包并在编译时链接
    
    ```
    <dependencies>
      <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.0</version>
      </dependency>
      ...
    </dependencies>
    ```
    
### Inheritance

- 如果是多模块的Maven项目，**parent POM**文件中应该加入`<packaging>pom</packaging>`，子模块会继承父模块的POM

- 类似与Java中的所有类继承自`java.lang.Object`类，Maven中所有POM文件也继承自**Super POM**，其中定义了Maven中央仓库的url、build的各种属性值等等

- Maven提供了`<dependencyManagement>`标签来管理依赖版本号，通常存在与**parent POM**中，子模块在引用时可以不指定版本号

- `<dependencyManagement>`里只是声明依赖，并不实现引入，而所有声明在`<dependencies>`里的依赖都会自动引入，并默认被所有的子项目继承

### Properties

- 定义属性值，使用时`${mysql.version}`这样引用

    ```
    <properties>
      <mysql.version>5.1.40</mysql.version>
      ...
    </properties>
    ```