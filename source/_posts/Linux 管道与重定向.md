---
title: Linux 管道与重定向
date: 2017-12-27T12:32:23.000Z
tags: ['Linux']
categories: [Ops]
---
## 标准 I/O 流

Linux shell 可以接收和发送字符序列或字符流形式的输入和输出。无论实际的字符流是来自还是来自文件，键盘，显示器上的窗口或其他IO设备，字符流都可以通过**文件IO**技术进行访问。Linux shell 使用3种**标准的I/O流**，每种流都与一个**文件描述符**。Linux 内核采用文件描述符（file descriptor）来访问文件。文件描述符是**非负整数**。打开现存文件或新建文件时，内核会返回一个文件描述符。读写文件也需要使用文件描述符来指定待读写的文件。

```
stdout   //标准输出流，它显示命令运行的输出，文件描述符为 1
stderr   //标准错误流，它显示命令运行的错误输出，文件描述符为 2
stdin    //标准输入流，它为命令提供输入，文件描述符为 0
```

输入流通常通过终端敲击键盘为程序提供输入。输出流通常向终端输出显示文本字符。

![linuxcom](/0.png)

## 重定向

在Linux命令行模式中，如果命令所需的输入不是来自键盘，而是来自指定的文件，这就是输入重定向。同理，命令的输出也可以不显示在屏幕上，而是写入到指定文件中，这就是输出重定向。

### > 符号

格式 `{stdout|stderr} > filename`，使用 `>` 符号将左侧输出重定向到一个文件中。需要注意的是：如果**文件名不存在则创建它，如果存在则覆盖**。

```sh
➜ echo "hello" > a.txt
➜ cat a.txt
hello
```

还可以这样：

```sh
➜ ls | grep "**.png" > a.txt
➜ cat a.txt 
arguments.png
mvvm.png
nrm-test.png
require.png
```

该命令将左侧命令的标准输出重定向到`a.txt`文件中。

特别的，使用 `/dev/null` 可以清空文件。`/dev/null`相当于Linux的黑洞文件，永远为空。

```sh
cat /dev/null > a.txt
```

### >> 符号

格式 `{stdout|stderr} >> filename`，`>>`符号和`>`符号作用差不多，如果文件不存在则创建，存在则**追加**内容到文件尾，不会覆盖。

```sh
➜ cat a.txt
hello
➜ echo "world" >> a.txt
➜ cat a.txt
hello
world
```

### < 符号

格式 `{command} < filename`，将文件的内容作为标准输入，传给左边的命令处理。

如下，将字符 `o` 替换为 `x`，但是源文件并不会改动：

```sh
➜ tr 'o' 'x' < a.txt
hellx
wxrld
➜ cat a.txt 
hello
world
```

又比如使用 `nc` 命令传输文件，发送端将文件作为标准输入传输给接受端，再由接收端写入文件。

```sh
nc -l {port} > filename // 接收端
nc {接受者IP} {port} < filename // 发送端
```

## 管道

管道命令操作符：`|` ，它能将前面一个命令传出的标准输出(stdout)，作为标准输入(stdin)传给下一条命令。相比于重定向，管道符号两边都是**可独立执行**的命令，通常用于命令间**传递参数**，而重定位符号的右侧必须是文件名。

![pipe](/1.png)

如下这条命令 `ls` 列出当前目录下所有非隐藏文件，作为标准输入传给 `grep`去过滤出以 `.png` 结尾的文件：

```sh
➜ ls | grep "**.png"
arguments.png
mvvm.png
nrm-test.png
require.png
```

### 参数传递

假设现在我启动了一个 Java 进程。

```sh
➜ ps -ef | grep java
  501  6292  6030   0 11:00PM ttys000    0:11.42 /usr/bin/java -jar /Users/s1mple/Library/Mobile Documents/com~apple~CloudDocs/Works/testvue/target/testvue-0.0.1-SNAPSHOT.jar
  501  6324  6224   0 11:05PM ttys001    0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn java
```

执行这条命令后终端打印出 Java 进程信息，进程号 6292。接下来我想结合管道命令 `kill` 掉这个 Java 进程。

```sh
➜ ps -ef | grep java | grep -v grep | awk '{print $2}' | xargs kill -9   
➜ ps -ef | grep java
  501  6352  6224   0 11:11PM ttys001    0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn java
```

这个命令看上去很长其实无非是多个命令组合在一起。我之所以想说这个是因为在**Tomcat**的运行脚本`catalina.sh`中看到了用于停止**Tomcat**的类似的命令。

```sh
➜ ps -ef | grep java | grep -v grep | awk '{print $2}' | xargs kill -9 
```

`ps -ef`列出所有进程传递给`grep java`过滤出包含 Java 关键字的进程，再传递给`grep -v grep`剔除包含**grep**关键字的进程，`awk '{print $2}'` 6292，再将参数传递给`xargs kill -9`最终杀掉进程。实际上等同于执行了 `kill -9 6292`。

很多时候比如说再写**shell**脚本时确实这样用，因为进程号是动态分配的，不能写死，只能通过一些手段动态获取进程号再进行**参数传递**。

## 命令替换

> 另外简单提一下**命令替换**。在Linux命令行模式下，当遇到一对 **\`\`** 时，将首先执行 **\`** 中间包含的命令，然后将其输出结果作为参数代入命令行中，这就是命令替换了。它**将一个命令的输出作为另外一个命令的参数**

过滤出`.*rc`的文件，将文件中的内容统一输出到`rc.txt`中

```
➜ cat `ls -a | grep ".*rc"` > ./Desktop/rc.txt
```

上诉命令和下面这条起一个作用

```
➜ ls -a | grep ".*rc" | xargs cat > ./Desktop/rc.txt
```

## 参考

- [Streams, pipes, and redirects](https://www.ibm.com/developerworks/linux/library/l-lpic1-103-4/index.html)
