---
title: 探究 HTTP 协议
date: 2017-12-25T01:39:59.000Z
tags: ['Web', 'Network']
categories: [Concepts]
---
## Preface

> 本文摘自油条的知乎文章 [摸索HTTP](https://zhuanlan.zhihu.com/p/23957087)，自己结合实验一步一步探究HTTP协议

## Prerequisites

- OS: MacOS或Linux各发行版
- `curl`、`nc`命令
- 多台电脑，最好是有自己的服务器

## nc

> **nc**(or **netcat**)命令用于建立TCP和UDP监听和连接，它可以建立TCP连接、发送UDP包、监听任意TCP和UDP端口、端口扫描和处理IPv4和IPv6等。它比**telnet**命令更加实用，在网络工具中有“瑞士军刀”的美誉。

- 打开两个终端。一个充当客户端，一个充当服务端。

- 让服务端`nc`启动之后监听8000端口，等待TCP连接
    ```
    nc -l 8000 //server
    ``` 
    
- 切换到客户端，如果是不同的机器则将`localhost`换成服务端主机的**IP**

    ```
    nc localhost 8000 //client
    ```
    
- 现在在任意一个终端中输入些字符，敲击回车，另一个终端会显示相应字符，说明双方已经成功建立TCP连接进行通信，需要注意的是一定要服务端先建立监听

#### Tips

- 可以直接用`nc`命令来传输文件

    ```
    nc -l {port} > filename // 接收端
    nc {接受者IP} {port} < filename // 发送端
    ```
    
- 亲测这条命令非常实用，适用于在你的本地和服务器未建立**ssh认证**时（不能使用**scp**），或内网网络传输文件，或你的两台电脑不能登陆同一微信传输文件时

## HTTP响应

> 浏览器在访问网站的时候，服务器到底给浏览器回复了一些什么玩意儿呢？有了上面的工具，接下来该探究一下HTTP了，到这里才正式开始切入正题了。首先，使用一下**curl**。**curl**能够向指定的URL发起一个http请求，并显示出请求回应中的内容。

- 先拿百度开刀，执行如下命令

    ```
    curl -i baidu.com
    ```
    
- 得到如下输出内容

    ```
    HTTP/1.1 200 OK
    Date: Mon, 25 Dec 2017 03:19:21 GMT
    Server: Apache
    Last-Modified: Tue, 12 Jan 2010 13:48:00 GMT
    ETag: "51-47cf7e6ee8400"
    Accept-Ranges: bytes
    Content-Length: 81
    Cache-Control: max-age=86400
    Expires: Tue, 26 Dec 2017 03:19:21 GMT
    Connection: Keep-Alive
    Content-Type: text/html

    <html>
    <meta http-equiv="refresh" content="0;url=http://www.baidu.com/">
    </html>
    ```
    这里，你所看到的这一大堆数据，便是一个完整的HTTP响应报文。也就是你在浏览器中输入网址 baidu.com 敲下回车之后，baidu服务器返回给浏览器的数据。
    
- **响应报文**包含如下部分：
    - 状态行
    - 响应头
    - 响应正文（可能没有）
    
    ![----](/0.png)

### 解析响应报文

> 了解HTTP响应的格式，以及其中的各部分字段的含义，有助于我们理解一些Web应用的功能以及浏览器的行为。

- 这里对HTTP响应的数据稍作分析。上述报文中的第一行：

    ```
    HTTP/1.1 200 OK
    ```
    为HTTP响应的状态，这里的200代表OK，表示服务器已经成功地处理了请求
    
- 报文往后的内容直到空行部分：

    ```
    Date: Fri, 25 Nov 2016 10:58:59 GMT
    Server: Apache
    Last-Modified: Tue, 12 Jan 2010 13:48:00 GMT
    ETag: "51-47cf7e6ee8400"
    Accept-Ranges: bytes
    Content-Length: 81
    Cache-Control: max-age=86400
    Expires: Sat, 26 Nov 2016 10:58:59 GMT
    Connection: Keep-Alive
    Content-Type: text/html
    ```
    这部分为HTTP响应的头部，其中大部分内容其实是可读性较强的，基本可以直接分析其中的含义。如`Content-Type: text/html`表示此次HTTP响应的结果为html文本。如果你访问的是一个图片，那么这里就可能是`Content-Type: image/png`了。头部中的`Content-Length: 81`表示整个响应的正文长度为81，这个也比较关键，告诉浏览器整个响应的正文部分数据有多少

- HTTP响应的头部中有许多非常有用的信息，这些信息往往决定着浏览器收到报文后接下来需要做出什么样的行动。也有一些非必须的内容，比如**Server、Accept-Ranges**

- 响应中的最后一部分内容，可以看到接下来页面刷新跳转到`http://www.baidu.com/`

    ```
    <html>
    <meta http-equiv="refresh" content="0;url=http://www.baidu.com/">
    </html>
    ```
    为响应的正文部分，这部分内容的长度是和响应头部的**Content-Length**对应值一致。
    
#### Verify Content Length

> 我们来验证正文长度是不是81。能用编程解决的就用编程解决，非常推荐平时用**python**或者**JavaScript**这种脚本语言去编程验证一些东西。只需要装个**node**或者**python**解释器就可以了，非常轻量级

- 这里用javascript，文件名a.js

    ```js
    // a.js
    
    const s = `<html>
    <meta http-equiv="refresh" content="0;url=http://www.baidu.com/">
    </html>
    `;
    console.log(s.length);
    ```
    其中`` ` ``反引号是ES6的新特性，可方便表示多行字符串。注意最后一行有回车
    
- 执行`node a.js`得到输出81

### 模拟HTTP响应

> 上面大致了解了HTTP响应的大体结构，那么，我们来自己构造一个合法的HTTP响应，看看浏览器能不能做出正常的反应。

- `nc -l 8000`开启一个服务端，监听8000端口，粘贴如下内容

    ```
    HTTP/1.1 200 OK
    Content-Length: 14
    Content-Type: text/html

    <h1>Hello</h1>
    ```

- 在浏览器端访问`http://localhost:8000/`，里面出现了一个大标题Hello

- 其实，浏览器在发起一个请求之后，只要服务器回应它一个符合HTTP标准的报文，它就能正常地解析并做出处理。而无论浏览器访问的对方是真的Web服务器还是假的nc，浏览器并不能知道它访问的目标到底是个什么东西，它只知道按照HTTP规范去解析收到的数据。这里可以看到，在网络中骗过浏览器是非常容易的，如传输的数据是未加密的，在这些传输的过程中很有可能会被拦截甚至篡改，浏览器被欺骗，进而用户被欺骗

## HTTP请求

> 在使用浏览器访问网站的时候，浏览器到底发送了一些什么东西过去呢？

- 其实我们就已经可以在终端里看到浏览器发送的请求报文

    ```
    GET / HTTP/1.1
    Host: localhost:8000
    Connection: keep-alive
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36
    Upgrade-Insecure-Requests: 1
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
    Accept-Encoding: gzip, deflate, br
    Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
    Cookie: Webstorm-f269b48=ebcfcf96-de94-4438-acc1-f68e40c6b6f1; session=eyJ1aWQiOjJ9.DR6wAg.ktPekLF--XP9XrDZlpbeT8J0XDE
    ```

- **请求报文**包含如下部分：
    - 请求行
    - 请求头
    - 请求体（可能没有）

    ![-----2](/1.png)


- 第一行为**请求行**，包含请求方法、请求路径和HTTP协议的版本三个信息。**请求头**通知服务器有关客户端请求的信息，比如上述头部中的**User-Agent**的值代表了浏览器信息，借助于此还可以做到根据不同的浏览器返回不同的页面（比如手机和PC的页面不一样）。最后多一行**回车换行**代表请求头的结束

- **GET请求**没有请求体，请求参数和对应的值附加在URL后面，利用问号`?`代表URL的结尾与请求参数的开始，与号`&`连接多个参数。传递参数**长度受限**，并且可以在地址栏清楚的看到，不适合传输私密数据及大量数据

- 而在填写登陆表单点击确认后一般都是发送**POST请求**，请求数据在请求体中，不会显示的展示在地址栏，但是还是可能被黑客获取，所以通常密码等敏感信息需要加密而不是明文传输

    ```
    POST /user/login HTTP/1.1
    Host: example.com
    Connection: keep-alive
    Upgrade-Insecure-Requests: 1
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.98 Safari/537.36
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
    Accept-Encoding: gzip, deflate, sdch, br
    Accept-Language: en-US,en;q=0.8,zh-CN;q=0.6,zh;q=0.4,zh-TW;q=0.2
    Content-Length: 27

    username=test&password=test
    ```
    
### 模拟HTTP请求

- 同样可以使用`nc`构造一个HTTP请求，`nc baidu.com 80`与百度服务器建立连接

    ```
    GET / HTTP/1.1
    Host: baidu.com
    ```
    
- 构造一个最精简的请求，敲击回车，终端上打印出响应报文

## Postscripts

> 实际上，按照前面的演示，在网络中，每一个应用都没法知道对面到底是个什么东西，只要双方相互收发的数据符合一定的格式和规则，它们之间就能够相互配合好工作。那么这些格式和规则，就时平时所谓的“协议”。