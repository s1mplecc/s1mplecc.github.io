---
title: MyBatis Generator
date: 2017-11-11T06:20:43.000Z
tags: ['Mybatis']
categories: [Coding]
---
## Preface

> MyBatis Generator(MBG)是适用于MyBatis和iBatis的代码生成器，它可以根据数据库自动生成Model层的JavaBean，DAO层的Mapper接口，映射SQL的XML文件。[官方文档](http://www.mybatis.org/generator/index.html)

- 我们先看一下项目结构，我们的目的是生成dao、model以及mapping目录下的代码
![-----2017-11-26---7.26.01](/0.png)

## Prerequisites

- Intellj IDEA
- Maven Project
- Spring Boot
- MySQL

## Maven Plugin

- 使用Maven我们可以简单快捷的导入MBG插件，并且通过Maven管理、启动插件。将下面内容加入到**pom.xml**中的`<build>`下的`<plugins>`标签中
    ```
    <plugin>
        <groupId>org.mybatis.generator</groupId>
        <artifactId>mybatis-generator-maven-plugin</artifactId>
        <version>1.3.5</version>
        <configuration>
            <configurationFile>${basedir}/src/main/resources/mybatis/generatorConfig.xml</configurationFile>
            <overwrite>true</overwrite>
            <verbose>true</verbose>
        </configuration>
        <dependencies>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>tk.mybatis</groupId>
                <artifactId>mapper</artifactId>
                <version>${mapper.version}</version>
            </dependency>
        </dependencies>
    </plugin>
    ```
    
#### Tips
1. **configurationFile**是xml配置文件的路径，**${basedir}** 是Maven内置变量项目根目录
2. **overwrite**表示覆盖原有文件，**verbose**表示MBG会将进度信息写入**build log**中，更多参数官网文档都有
3. 使用了两个依赖，**mysql-connector**用于连接数据库，**tk.mybatis.mapper**是比较好用的通用Mapper插件

## generatorConfig.xml

[详解MBG配置文件](http://blog.csdn.net/isea533/article/details/42102297)

- 最关键的应该就算MBG的配置文件了。这个文件告诉MBG怎么去连接数据库、用到哪些表、生成哪些对象等等

    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE generatorConfiguration
            PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
    <generatorConfiguration>
        <properties resource="mybatis/config.properties"/>
        <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
            <property name="beginningDelimiter" value="`"/>
            <property name="endingDelimiter" value="`"/>

            <property name="javaFormatter" value="org.mybatis.generator.api.dom.DefaultJavaFormatter"/>
            <property name="xmlFormatter" value="org.mybatis.generator.api.dom.DefaultXmlFormatter"/>

            <plugin type="${mapper.plugin}">
                <property name="mappers" value="${mapper.Mapper}"/>
            </plugin>
            <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>

            <jdbcConnection driverClass="${jdbc.driverClass}"
                            connectionURL="${jdbc.url}"
                            userId="${jdbc.user}"
                            password="${jdbc.password}">
            </jdbcConnection>

            <javaModelGenerator targetPackage="info.s1mple.webscraping.model" targetProject="src/test/java"/>

            <sqlMapGenerator targetPackage="mybatis.mapping" targetProject="src/test/java"/>

            <javaClientGenerator targetPackage="info.s1mple.webscraping.dao" targetProject="src/test/java" type="XMLMAPPER"/>

            <table tableName="ws_record" domainObjectName="Record">
                <generatedKey column="record_id" sqlStatement="Mysql" identity="true"/>
            </table>
        </context>

    </generatorConfiguration>
    ```
    
#### Tips
1. `<properties>`用于指定外部配置文件，使用相对路径，可以使用`${}`进行引用
2. 至少含有一个`<context>`标签。其中`targetRuntime`默认为**MyBatis3**，**MyBatis3Simple**可以不生成Example查询有关内容，**flat**为每一张表生成一个实体类
3. `<property>`配置一些属性，`beginningDelimiter`和`endingDelimiter`将表的字段用单引号扩起来，`javaFormatter`和`xmlFormatter`格式化生成的代码
4. `<plugin>`定义插件用于扩展或修改MBG生成的代码。我使用了Mapper插件，用于修改生成的Mapper接口，以及序列化插件，用于修改Model类继承**Serializable**接口
5. `<jdbcConnection>`连接数据库，配置我都写在了**config.properties**中方便管理
6. `<javaModelGenerator`生成Model类，`targetPackage`是生成的类package，`targetProject`生成到哪个路径下，建议都放在test文件夹而不是直接覆盖源代码。
7. `<sqlMapGenerator>`生成Mapper.xml文件
8. `<javaClientGenerator>`生成Mapper接口
9. `<table>`使用到的表，可以设置对象名，自增主键等

## Run It

- 可以命令行执行`mvn mybatis-generator:generate`运行MBG，或者直接在Maven Plugins中双击运行
![-----2017-11-27---10.37.48](/1.png)

## Results Show

- **生成的Model类**，可以看到package就是我们在`targetPackage`中的写的内容，有各种注解及注释信息，以及自增主键的注解。
![-----2017-11-27---1.15.04](/2.png)

- **生成的Mapper.xml**，一般还要自己根据需求添加SQL语句
![-----2017-11-27---1.24.34](/3.png)

- **生成的Dao层的Mapper接口**，BaseMapper是自己写的继承了tk mapper的接口类。还需要根据需求添加方法
![-----2017-11-27---1.33.29](/4.png)
    
## Postscripts

> 一次配置，多次受益，大大减少了重复性、无脑的代码的编写，何乐而不为呢？