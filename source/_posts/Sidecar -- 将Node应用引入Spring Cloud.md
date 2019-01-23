---
title: Sidecar -- 将 Node 应用引入 Spring Cloud
date: 2018-03-31T02:05:08.000Z
tags: ['Spring Cloud', 'Node.js']
categories: [Coding]
---
## Preface

> Sidecar 起源于 [Netflix Prana](https://github.com/Netflix/Prana)，它的目的是将 **Non-JVM** 语言整合到 Netflix OSS 生态系统中，如今 Spring Cloud 将 Spring Boot 与 Netflix OSS 整合成一套微服务解决框架，大大简化了程序员的开发。而 Sidecar 就是其中的一个衍生物，用于将 Non-JVM 语言，譬如 Node.js、Python 引入至 Spring Cloud 框架中。

- 本文的示例代码已上传至 [Github](https://github.com/s1mplecc/sidecar-example)

- 使用 Maven 构建的多模块项目`sidecar-example`，并将 Node.js 开发的`book-service`模块拷贝至一个项目中

    ```
    // 模块结构如下
    ─ sidecar-example
       ├── author-service   // Java 开发的微服务，用于测试服务间通信
       ├── eureka-server    // 注册中心
       ├── node-sidecar    // Sidecar，用于将 Node 应用引入 Spring Cloud
       └── book-service    // Node.js 开发的微服务
    ```

- 使用的 Spring Boot 版本及 Spring Cloud 全家桶版本

    ```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath/>
    </parent>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-parent</artifactId>
                <version>Edgware.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    ```

## Sidecar

> 先来看看引入 Sidecar 需要做些什么
 
- **添加 Maven 依赖**，很简单，甚至于不需要`eureka-client`的依赖，因为它已经整合至 Sidecar 的依赖中

    ```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix-sidecar</artifactId>
        </dependency>
    </dependencies>
    ```
    
- 接下来是**注解**，在 Sidecar 主类上添加`@EnableSidecar`注解，我们来看看这个注解包含些什么

    ```java
    @EnableCircuitBreaker
    @EnableZuulProxy
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Import(SidecarConfiguration.class)
    public @interface EnableSidecar {

    }
    ```
    包含了**网关 Zuul** 以及微服务结构中不可或缺的**熔断器 Hystrix**
    
- 最后是**配置文件**，在`application.yml`中添加如下配置

    ```yml
    server:
      port: 9091
    spring:
      application:
        name: node-sidecar
    eureka:
      instance:
        hostname: localhost
      client:
        serviceUrl:
          defaultZone: http://${eureka.instance.hostname}:9090/eureka/
    sidecar:
      port: 3000
      health-uri: http://localhost:${sidecar.port}/health
    ```
    声明服务名和注册中心地址都没什么问题，最核心的就是 sidecar 的几个配置，包括
    - `sidecar.port` 监听的 Node 应用的端口号，
    - `sidecar.health-uri` Node 应用的健康检查接口的 uri

#### 健康检查接口

> Node.js 的微服务应用`book-service`我采用的 **Koa** 框架，这里不多赘述，需要注意的是：该应用必须实现一个`/health`健康检查接口，Sidecar 应用会每隔几秒访问一次该接口，并将该服务的健康状态返回给 Eureka

- 只需要返回`{ status: 'UP' }`这样一串 Json 即可

    ```
    const Koa = require('koa')
    const router = require('koa-router')()

    const app = new Koa()

    // log request URL:
    app.use(async (ctx, next) => {
      console.log(`Process ${ctx.request.method} ${ctx.request.url}...`)
      await next()
    })

    // add routes:
    router.get('/health', async (ctx, next) => {
      ctx.response.body = {
        status: 'UP'
      }
    })

    // add router middleware:
    app.use(router.routes())

    app.listen(3000)
    console.log('app started at port 3000...')

    ```
    
## 服务注册

> 到目前为止，Sidecar 和 Node 服务已准备就绪，我们按顺序启动 **eureka-server** => **book-service** => **node-sidecar**
 
- 访问 Eureka 的 WebUI `http://localhost:9090/`，Sidecar 已被注册到 Eureka 上，并且状态为 UP
![B89413D5-130D-4F5D-BB76-7559B6B0A036](/0.png)如果我们停掉`book-service`，则`node-sidecar`的服务状态会变为 Down，显示服务不可用

- 访问 Sidecar 的首页`http://localhost:9091/`，提供了三个接口
![A2B5DA24-46A6-40B9-9129-54EC7FA0F329](/1.png)
    
- 访问`/health`可以得到 Node 应用返回的健康报告
    ```json
    {
        "status": "UP"
    }
    ```
    访问`hosts/node-sidecar`可以的到 Sidecar 实例的一些信息
    ```json
    [
        {
            "host": "localhost",
            "port": 3000,
            "metadata": {},
            "instanceInfo": {
                "instanceId": "192.168.1.8:node-sidecar:9091",
                "app": "NODE-SIDECAR",
                "appGroupName": null,
                "ipAddr": "192.168.1.8",
                "sid": "na",
                "homePageUrl": "http://localhost:3000/",
                "statusPageUrl": "http://localhost:9091/info",
                "healthCheckUrl": "http://localhost:9091/health",
                "secureHealthCheckUrl": null,
                "vipAddress": "node-sidecar",
                "secureVipAddress": "node-sidecar",
                "countryId": 1,
                "dataCenterInfo": {
                    "@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
                    "name": "MyOwn"
                },
                "hostName": "localhost",
                "status": "UP",
                "leaseInfo": {
                    "renewalIntervalInSecs": 30,
                    "durationInSecs": 90,
                    "registrationTimestamp": 1522469074848,
                    "lastRenewalTimestamp": 1522469344919,
                    "evictionTimestamp": 0,
                    "serviceUpTimestamp": 1522469074848
                },
                "isCoordinatingDiscoveryServer": false,
                "metadata": {},
                "lastUpdatedTimestamp": 1522469074849,
                "lastDirtyTimestamp": 1522469044798,
                "actionType": "ADDED",
                "asgName": null,
                "overriddenStatus": "UNKNOWN"
            },
            "secure": false,
            "uri": "http://localhost:3000",
            "serviceId": "NODE-SIDECAR"
        }
    ]
    ```
    可以看到，该实例维护了 Node 应用的访问地址`"uri": "http://localhost:3000"`，这也是接下来要说的：其他微服务可以通过 Sidecar 的服务名声明式调用 Node 服务
    
## 声明式服务调用

> 以上，我们已经验证了 Eureka 可以通过 Sidecar 间接的管理基于 Node 的微服务。而在微服务体系中，还有非常重要的一点，就是服务间的调用。Spring Cloud 允许我们使用**服务名**进行服务间的调用，摒弃了原先的固定写死的 IP 地址，便于服务集群的横向拓展及维护。那么，Non-JVM 的微服务与其他服务间是否可以通过服务名互相调用呢，答案是可以的。
 
### 被调用

> 我们假设下面一个场景，book-service 提供了根据 bookId 返回对应 book 信息的接口，其中包含 authorId，而 author-service 需要根据该 authorId 获取到对应的作者信息。也就是 author-service 需要访问 book-service 的`/book/:bookId`接口拿到 authorId
 
- 先在 book-service 中实现`/book/:bookId`接口，先固定返回 authorId 为 1

    ```js
    router.get('/book/:bookId', async (ctx, next) => {
      ctx.response.body = {
        bookId: ctx.params.bookId,
        authorId: "1",
        description: "This is a book."
      }
    })
    ```

- 而在 author-service 中声明式服务调用使用的是 Spring Cloud 整合的 **Feign**，Maven 依赖如下

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
    </dependency>
    ```
    
- author-service 作为 book-service 服务的调用者，需要声明 Client 接口，代码如下

    ```java
    @FeignClient("node-sidecar")
    public interface BookServiceClient {

        @GetMapping("/book/{bookId}")
        Book getBook(@PathVariable("bookId") String bookId);

    }
    ```
    注意到`@FeignClient`注解中调用的服务名填写的是 node-sidecar (大小写不敏感)，因为自始至终 Eureka 中注册的是 Sidecar 的信息，而 Sidecar 实例维护了 book-service 的地址信息，所以它可以将请求转发至 book-service
    
- 在 author-service 中提供`/book/{bookId}/author`接口

    ```java
    @RestController
    public class AuthorController {

        @Resource
        private BookServiceClient bookServiceClient;

        @GetMapping("/book/{bookId}/author")
        public Author getAuthor(@PathVariable String bookId){
            Book book = bookServiceClient.getBook(bookId);
            return new Author(book.getAuthorId(),"Jack");
        }

    }
    ```
    
- 启动 author-service ，访问`http://localhost:9092/book/1/author`

    ```json
    {
        "authorId": "1",
        "authorName": "Jack"
    }
    ```
    
### 调用其他微服务

> 其他微服务可以通过 Sidecar 实例的服务名间接调用基于 Node 的 book-service。同样的，book-service 也可以通过服务名调用其他微服务，这要归功于`@EnableZuulProxy`

- 访问`http://localhost:9091/author-service/book/1/author`惊讶的发现，这和我们访问`http://localhost:9092/book/1/author`结果是一样的，这是由于 Sidecar 引入了 Zuul 网关，它可以获取 Eureka 上注册的服务的地址信息，从而进行路由跳转
![A44BC958-7050-4530-8855-7F0042A6C32F](/2.png)

> **Tips:** Spring Cloud Zuul 通过与 Eureka 的整合，将自身注册为 Eureka 服务治理下的应用，同时从 Eureka 获取其他所有微服务的示实例信息。对于路由规则的维护，Zuul 默认会将服务名作为 ContextPath 的方式创建路由映射

- 那接下来就简单了，先在 author-service 中添加一个接口返回作者描述

    ```java
    @GetMapping("/author/{authorId}")
    public String getAuthorDescription(@PathVariable String authorId) {
        return "This is an author description of " + authorId;
    }
    ```

- 再在 book-service 中添加 book 的详细信息接口，需要去调用上述的作者描述接口

    ```js
    const axios = require('axios')
    const SIDECAR_URI = 'http://localhost:9091'
    const AUTHOR_SERVICE = 'author-service'
    
    router.get('/book/:bookId/detailed', async (ctx, next) => {
      await axios.get(`${SIDECAR_URI}/${AUTHOR_SERVICE}/author/1`).then(res => {
        ctx.response.body = {
          bookId: ctx.params.bookId,
          authorId: '1',
          authorDescription: res.data,
          description: 'This is a book.'
        }
      }).catch(error => console.log('error', error))
    })
    ```
    这里采用的 author-service 服务名去访问的该服务，而不是固定的 IP 地址，格式为`${SIDECAR_URI}/${AUTHOR_SERVICE}`
    
- 访问`http://localhost:3000/book/1/detailed`

    ```json
    {
        "bookId": "1",
        "authorId": "1",
        "authorDescription": "This is an author description of 1",
        "description": "This is a book."
    }
    ```
    
## Api-Gateway

> 后来我又加入了`api-gateway`去验证网关能否通过`node-sidecar`的服务名去将访问 API 的请求转发到`book-service`，事实证明确实是可以的
 
- `api-gateway`的 Maven 依赖

    ```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
        </dependency>
    </dependencies>
    ```
    
- 需要在主类上加上`@EnableZuulProxy` Zuul 网关和`@EnableDiscoveryClient`注解
 
- 配置文件

    ```yml
    spring:
      application:
        name: api-gateway
    server:
      port: 5555
      context-path: /
    eureka:
      instance:
        hostname: localhost
      client:
        serviceUrl:
          defaultZone: http://${eureka.instance.hostname}:9090/eureka/
    zuul:
      routes:
        author-service:
          path: /author-service/**
          serviceId: author-service
        book-service:
          path: /book-service/**
          serviceId: node-sidecar
    ```  
    
- 访问`http://localhost:5555/book-service/book/1`，在接口层面，`node-sidecar`其实是不可见的，访问的还是`book-service`的 API
![WX20180403-110158](/3.png)  

- 由此，如果`book-service`需要横向拓展为一个集群，无非是多创建几个对应的 Sidecar 应用，它们的服务名只要是一样的，那就可以通过网关去实现负载均衡，甚至我第二个`book-service`用 Python 写也没有问题

## Conclusion

> 由此，基于 Non-JVM 语言的微服务，通过 Sidecar ，实现了服务向 Eureka 的注册与健康检查，并且可以通过服务名进行服务间调用。基本上可以说 Non-JVM 应用通过 Sidecar 完美的融入了 Spring Cloud 生态圈。
 
- 附上一张架构图
![sidecar](/4.png)
    
## Preferences

- [使用 Sidecar 将 Node.js 引入 Spring Cloud](https://github.com/marshalYuan/spring-cloud-example/blob/master/docs/sidecar.md)