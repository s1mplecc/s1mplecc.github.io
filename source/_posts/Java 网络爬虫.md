---
title: Java 网络爬虫
date: 2018-11-05T13:15:21.000Z
tags: ['Java']
categories: []
---
## Preface

很早之前写过一个 Java 的爬虫程序，可以爬取笔趣阁上的小说。最近突然有爬点书看看的想法，遂翻到原先的代码，简直不忍直视。好在现在编码能力有所提升，而且团队内一直强调重构的重要性，所以重新拾起这个爬虫程序，好好重构一番。代码已上传至 [GitHub](https://github.com/s1mplecc/web-crawler)

本文主要讲讲：
1. 使用 **Gradle** 构建项目时遇到的问题以及解决办法
2. 使用 JDK8 的特性如何简便地进行**并行开发**
3. 写爬虫时的思路

## Gradle

不得不说，Gradle 让我眼前一亮，相比 Maven，它抛弃了冗余繁琐的 XML 格式，使用语法精炼的 **Groovy**，以下就是我的项目构建文件 **build.gradle** 的全部内容，相当于 Maven 的 pom.xml，清爽简短但重要的信息一点不少。

```js
plugins {
    id 'java'
}

group 'xyz.s1mple.crawler'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

repositories {
    mavenLocal()
    mavenCentral()
}

dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.3.1'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.3.1'
    testCompile 'org.assertj:assertj-core:3.11.1'
    compile 'org.slf4j:slf4j-api:1.7.25'
    runtime 'org.slf4j:slf4j-simple:1.7.25'
}

test {
    useJUnitPlatform()

    testLogging {
        events "passed", "skipped", "failed"
    }
}
```
在上面的配置文件中我们可以发现，Gradle 沿用了 Maven 的中央仓库，同时还可以设置 Maven 本地仓库，同时 Gradle 项目生成的构件也可以发布到 Maven 仓库供他人使用，这一点也是支撑 Gradle 快速发展的重要一点。

#### 安装

关于安装，我使用的 IDE 是 IntelliJ IDEA，使用内置的 Gradle 也可，但最好是下载安装全局的 Gradle，这样可以随时随地跑 Gradle 命令。Mac 下安装十分方便。其他系统如何安装可以在 [Gradle官网](https://gradle.org/install/#with-a-package-manager) 上找到。

```sh
brew install gradle
```

#### JUnit5

在使用 Gradle 的时候我遇到了一个问题：明明添加了 JUnit5 也就是 jupiter 的依赖，为什么使用 `gradle test` 命令没用任何输出。这个问题在我将依赖换回 Junit4 时得到解决，也就是说果然是配置的问题。在 build.gradle 中添加如下配置即可。

```js
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.3.1'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.3.1'
}
    
test {
    useJUnitPlatform()
}
```
还可以添加 `testLogging`。再运行命令有如下输出
```
➜  web-crawler git:(master) ✗ gradle clean test

> Task :test

xyz.s1mple.crawler.HtmlParserTest > should_parse_novel_content_from_html() PASSED

xyz.s1mple.crawler.HtmlParserTest > should_parse_novel_title_from_html() PASSED

xyz.s1mple.crawler.HtmlParserTest > should_parse_chapter_uris_from_html() PASSED

BUILD SUCCESSFUL in 1s
```

## 并行

Java 多线程编程是越来越容易了，从最早的 Thread，Runnable 到 JDK5 的ExecutorService 到 JDK7 的 ForkJoin 框架。现在 JDK8 又提供了**并行流**（**parallelStream**）来简化这一过程。

```java
List<Integer> cost = Lists.newArrayList(1, 3, 7, 9, 34);
long total = cost.parallelStream()
                    .map(x -> x += 1)
                    .reduce((y, z) -> y + z)
                    .get();
```

在 parallelStream 中默认线程池大小是机器的 CPU 核数。**默认情况下，所有的 Fork/Join 任务都会共用同一个线程池，线程的数量等于CPU的核数**。所以在未手动设置线程池数量的情况下我的电脑会启动 4 个线程，而爬虫的瓶颈在于网络 I/O 不在 CPU，所以果断自定义线程数。效果也十分显著，爬取两千多章的时间从原先的两分多钟变为 20s。
	
```java
final ForkJoinPool pool = new ForkJoinPool(PARALLELISM_LEVEL);
pool.submit(() -> urls.parallelStream()
        .map(url -> htmlParser.parseContent(urlReader.read(url)))
        .reduce((x, y) -> x + y)
        .orElse(""))
        .get();
```
需要注意两点：
1. `pool.submit()` 该方法为**懒加载**，如果**不调用它的结果则实际不会执行**，最后 `.get()` 获取执行结果。
2. `.reduce((x, y) -> x + y)` 这一步貌似会有很多字符串拼接影响效率，实则底层会使用 `StringBuilder` 或 `StringBuffer` 帮你做了性能优化

#### 测试

关于并行数对爬虫程序的性能影响，实际测试时受网络波动的影响，测试数据可能波动较大，连接每个 Url 的耗时在 50ms ～ 1000ms 波动区间。为减少网络不稳定带来的影响，以下测试分为 100 章、1000 章两个数量级，进行不同并行等级的循环测试，测试 3 轮，时间单位为秒。

**数量级：103 章**
| PARALLELISM_LEVEL | 1 | 4 | 16 | 64 | 256 |
|:--:|:--:|:--:|:--:|:--:|:--:|
| 第1次测试 |9|5|2|2|2|
| 第2次测试 |8|4|2|2|2|
| 第3次测试 |9|3|2|2|3|

**数量级：1180 章**
| PARALLELISM_LEVEL | 4 | 8 | 16 | 32 | 64 | 128 | 256 |
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 第1次测试 |28|16|10|13|12|12|11|
| 第2次测试 |23|16|14|11|12|12|14|
| 第3次测试 |25|29|14|13|29|20|20|

## 爬虫思路

最后说一说自己写这个爬虫时的思路，也算是解决问题的入手点。主要的解析类为 HtmlParser ，负责解析 Html。其实这个程序就是通过 Html 源码解析小说名 Title、小说所含所有章节的 Url、小说每一章的内容，最后将内容 reduce 写入文件。

#### Title

解析小说章节目录网址 Html 源码的 `<h1>...</h1>` 标签中的值

```
// 章节目录网址
http://www.biquge.cm/9/9422/

// html
<h1>大道朝天</h1>

// after parse
大道朝天
```

#### Chapter URIs

解析小说章节目录网址 Html 源码的 `<div id="list">...</div>` 中包含的 `<a href="...">`

```
// 章节目录网址
http://www.biquge.cm/9/9422/

// html
<div id="list">
<dl><dd><a href="/9/9422/6927857.html">第一章 三千里禁</a></dd>
<dd><a href="/9/9422/6927858.html">第二章 斩天一剑</a></dd>
<dd><a href="/9/9422/6927859.html">第三章 再次踏进那条河的白衣少年</a></dd>
...
</dl></div>

// after parse
["/9/9422/6927859.html", "/9/9422/6927858.html", "/9/9422/6927857.html" ...]
```

#### Content

根据 Chapter URIs 解析每一章的内容，最后拼接在一起写入文件。根据 `<div id="content">...</div>` 解析，`&nbsp;` 替换为空格，`<br />` 替换为换行符

```
// 某一章的网址
http://www.biquge.cm/9/9422/6927857.html

// html
<div id="content">&nbsp;&nbsp;&nbsp;&nbsp;朝天大陆南方，一片青山绵延数千里，数百秀峰终年隐在云雾中。<br />
<br />
&nbsp;&nbsp;&nbsp;&nbsp;天下第一修行大派青山宗便在此间，普通人极难一睹真容。<br />
<br />
...
</div>

// after parse

    朝天大陆南方，一片青山绵延数千里，数百秀峰终年隐在云雾中。
    
    天下第一修行大派青山宗便在此间，普通人极难一睹真容。
```

## References

- [Gradle，构建工具的未来？](http://www.infoq.com/cn/news/2011/04/xxb-maven-6-gradle)
- [How to use JUnit 5 with Gradle](https://medium.com/@jonashavers/how-to-use-junit-5-with-gradle-fb7c5c3286cc)