---
title: Spring Cloud 集成 GitHub 第三方认证
date: 2018-04-13T07:23:14.000Z
tags: ['Spring', 'Security']
categories: [Coding]
---
## Preface

> 最近遇到需要在 Spring Cloud 框架中加入**用户认证**的功能，发现 GitHub 有第三方认证功能，遂先写了个 demo 实现通过 GitHub 三方认证完成用户认证，虽然不是最终解决方案，但是有助于理解用户认证的流程
 
- 代码沿用之前的 [Sidecar -- 将Node应用引入Spring Cloud](https://s1mple.online/sidecar/) 博客中的代码，用到以下模块

    ```
    ─ sidecar-example
       ├── author-service    // 需要认证才能访问的微服务
       ├── eureka-server    // 注册中心
       └── api-gateway    // 网关，一切访问的入口
    ```
 
> 主要修改的是`api-gateway`，实现效果就是：**用户通过`api-gateway`访问任何一个微服务，需要先跳转到 GitHub 进行认证，认证通过后才获取访问权限**

## Concepts

在这之前，有一些概念需要先了解

### SSO

> **SSO**(or **Single Sign On**) **单点登录**，是目前比较流行的企业业务整合的解决方案之一。SSO 的定义是在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统。非常适用于微服务架构，可以实现**在一个微服务应用登录后（通常有专门的认证服务器），即可访问该微服务集群中的所有其他微服务**。
 
### OAuth 2.0

> 大家平时肯定遇到过第三方登录的情况，不知道有没有仔细想过它的流程。假设现在我们用微博账号登录简书

- 最传统的办法是让用户直接在简书的登录页面输微博的账号和密码，简书通过用户的账号和密码去微博那里获取用户数据，但这样做有很多严重的缺点：

    ```txt
    简书需要保存用户的微博账号和密码，这样很不安全。
    简书拥有了获取用户在微博所有的权限，包括删除好友、给好友发私信、更改密码、注销账号等危险操作。
    用户只有修改微博密码，才能收回赋予简书的权限。但是这样做会使得其他所有通过微博登录的第三方应用程序全部失效。
    只要有一个第三方应用程序被破解，就会导致用户密码泄漏，以及所有使用微博登录的网站的数据泄漏。
    ```
    
- 为了解决以上的问题，OAuth 协议应运而生。我们来看一下它的流程

    ```txt
    简书问新浪微博：我想要获取用户 A 的头像和昵称，请你提供
    微博说：我需要经过用户Ａ 本人的许可 => 然后微博询问用户 A 是否要授权简书访问自己的头像和昵称
    用户授权后，微博对简书说：我给你一个临时钥匙（Token），你访问我带了这把钥匙，我就把用户的资料给你
    ```

- 以上流程可由下图表示
![291523855245_.pic](/0.jpg)

- 其中最关键的两个部分如下

    ```
    ② 用户同意授权给客户端，则认证服务器返回 Code，即授权码
    ③ 客户端带着 Code，去认证服务器申领 Token
    ```
    
---
#### 授权码模式

> 以上模式称为授权码模式 (**authorization code**) ，是目前功能最完整、流程最严密的客户端授权模式。它的特点就是通过客户端服务器，与"服务提供商"的认证服务器进行互动

- 步骤如下

    ```txt
    用户访问客户端，后者将前者导向认证服务器。
    认证服务器询问用户是否给予客户端授权。
    若用户给予授权，认证服务器将用户导向客户端事先指定的"重定向URI"（redirection URI），同时附上一个授权码。
    客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。
    认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。
    ```
    ![bg2014051204](/1.png)

## Register

> 接下来进入正题，如何基于 GitHub 进行第三方认证。首先，我们需要先前往 GitHub 注册一个新的 OAuth App
 
- 前往`https://github.com/settings/developers`点击`Register a new application`，出现如下界面，填写相应信息。注意，主页和回调的 URL 填写的都是`api-gateway`的地址
![B2241479-07A7-4E03-BB62-BCA4CDA5A4A3](/2.png)

- 注册成功后生成的`Client ID`和`Client Secret`是后续需要填写到配置文件中
![C65DDEFD-F282-4F12-A211-938767ECD7DA](/3.png)

## Coding

> 接下来就是编码`api-gateway`实现重定位到刚注册的 OAuth App 进行认证

- 添加 Maven 依赖

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-oauth2</artifactId>
    </dependency>
    ```
    
- 在主类上添加`@EnableOAuth2Sso`注解

    ```java
    @SpringBootApplication
    @EnableDiscoveryClient
    @EnableZuulProxy
    @EnableOAuth2Sso
    public class ApiGatewayApplication {

        public static void main(String[] args) {
            SpringApplication.run(ApiGatewayApplication.class, args);
        }
    }
    ```
    
- 为了测试过滤情况，我加入了`/`和`/test`接口，除此之外，还有`/user`接口返回 GitHub 个人信息的接口

    ```java
    @RestController
    public class Controller {

        @GetMapping("/")
        public String welcome() {
            return "welcome";
        }

        @GetMapping("/test")
        public String test() {
            return "test";
        }

        @RequestMapping("/user")
        public Principal user(Principal user) {
            return user;
        }
    }
    ```

- 修改`application.yml`

    ```yml
    spring:
      application:
        name: api-gateway
    server:
      port: 5555
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

    security:
      ignored: /,/test  # 认证忽略的接口
      sessions: never   # session 策略
      oauth2:
        sso:
          loginPath: /login   # 登录路径
        client:
          clientId: 554906e366f7861eab21
          clientSecret: e1286e1ac17599c7a8638bb24649e79f5aabddfd
          userAuthorizationUri: https://github.com/login/oauth/authorize   # 进行用户授权的路径
          accessTokenUri: https://github.com/login/oauth/access_token   # 获取 token 的路径
        resource:
          userInfoUri: https://api.github.com/user
          preferTokenInfo: false
    ```
    
## Test

> 现在就可以进行测试了。陆续启动`eureka-server`、`api-gateway`、`author-service`三个微服务
 
- 访问`localhost:5555/`和`localhost:5555/test`接口，发现均不需要认证，即`security.ignored: /,/test`配置生效
![1C21C2B8-E689-4BBC-94EE-4ED74796994C](/4.png)

- 接下来通过网关访问`author-service`的接口，正常需要通过 GitHub 认证，事实证明确实如此。访问`http://localhost:5555/author-service/author/1`接口，跳转到**用户授权**界面
![D2FCA844-1A95-4907-A350-68A21ACCE38C](/5.png)

- 确认授权后重定位到`/author-service/author/1`接口返回数据
![ED7EE656-F376-408D-9DB8-2C6908799A07-1](/6.png)

- 还可以访问`http://localhost:5555/user`来获取 GitHub 账号的信息，这个就不贴图了，自己测试即可

- 再回到 GitHub 的 OAuth 界面，发现`0 user`变成`1 user`了，在此你可以弃用所有的用户 Token 以及重置 Client Secret 密钥
![7E65EF94-3B65-4FE3-B814-29C6B8A54ECF](/7.png) 

## Analyze

> 以下分析主要是为了看清重定位、URL 的参数和`application.yml`中的配置有何关联

- 访问`http://localhost:5555/author-service/author/1`后，重定位到`http://localhost:5555/login`
![D33AA927-F963-41C7-BE5F-6AA7C4FF09AF](/8.png)

- 访问`http://localhost:5555/login`重定位`https://github.com/login/oauth/authorize`即 GitHub 的用户授权路径
![E652C938-12A3-4A56-9367-0D07D76B1B66](/9.png)

- `/login/oauth/authorize`这个界面才是用户看到的需要点击授权的页面，我们来看下 URL 携带的参数：

    ```
    client_id: 554906e366f7861eab21
    redirect_uri: http://localhost:5555/login
    response_type: code
    state: z2MaR7
    ```

- 用户点击确认授权后，注意此 Token 非彼 Token
![71F69537-2DE4-49BD-AC22-774211576934](/10.png)

- 接下来带着 code 和 state 访问`http://localhost:5555/login`再重定位到`http://localhost:5555/author-service/author/1`真正想访问的接口
![05DAA887-01B1-42F4-9C20-0820777E7D92](/11.png)

> 事实上，由于申领 Token 这一步是在客户端的后台服务器上完成的，所以对用户不可见。其实客户端在用户点击授权后向`https://github.com/login/oauth/access_token`发送了一个 POST 请求，并带上`client_id`、`client_secret`、上一步获取的`code`以及`redirect_uri`等参数，由此来获取`access_token`

## References

- [Spring Cloud Security - 官网](https://cloud.spring.io/spring-cloud-security/)
- [Authorization options for OAuth Apps](https://developer.github.com/apps/building-oauth-apps/authorization-options-for-oauth-apps/)
- [理解 OAuth 2.0 - 阮一峰](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
- [OAuth 认证流程详解](https://www.jianshu.com/p/0db71eb445c8)
- [Spring Cloud Security 集成 CAS 对微服务认证](https://segmentfault.com/a/1190000011098539)