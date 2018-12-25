---
title: Linux流、管道&重定向
date: 2017-12-27T12:32:23.000Z
tags: ['Linux']
categories: []
---
## 标准I/O流

> **Linux shell**（比如**Bash**）可以接收和发送字符序列或字符流形式的输入和输出。无论实际的字符流是来自还是来自文件，键盘，显示器上的窗口或其他IO设备，字符流都可以通过**文件IO**技术进行访问

- **Linux shell**使用3种**标准的I/O流**，每种流都与一个**文件描述符**
    
    ```c++
    1.stdout   //标准输出流，它显示命令运行的输出，文件描述符为 1
    2.stderr   //标准错误流，它显示命令运行的错误输出，文件描述符为 2
    3.stdin    //标准输入流，它为命令提供输入，文件描述符为 0
    ```

- 输入流通常通过终端敲击键盘为程序提供输入。输出流通常向终端输出显示文本字符

- Linux内核采用文件描述符（**file descriptor**）来访问文件。文件描述符是**非负整数**。打开现存文件或新建文件时，内核会返回一个文件描述符。读写文件也需要使用文件描述符来指定待读写的文件

- 一条Linux命令执行的流程图
![linuxcom](/0.png)

---

## 重定向

> 在Linux命令行模式中，如果命令所需的输入不是来自键盘，而是来自指定的文件，这就是输入重定向。同理，命令的输出也可以不显示在屏幕上，而是写入到指定文件中，这就是输出重定向

### " > "

- 格式`{stdout|stderr} > filename`

- 使用`>`符号将左侧输出重定向到一个文件中

    ```
    ➜ echo "hello" > a.txt
    ➜ cat a.txt
    hello
    ```
    该命令将`hello`添加到`a.txt`中，如果文件不存在则创建它，如果存在则**覆盖**
    
- 还可以这样

    ```
    ➜ ls | grep "**.png" > a.txt
    ➜ cat a.txt 
    arguments.png
    mvvm.png
    nrm-test.png
    require.png
    ```
    该命令将左侧命令的标准输出重定向到`a.txt`文件中
    
- 使用`/dev/null`清空文件

    ```
    cat /dev/null > a.txt
    ```
    `/dev/null`相当于Linux的黑洞文件，永远为空

### " >> "

- 格式`{stdout|stderr} >> filename`

- `>>`符号和`>`符号作用差不多，如果文件不存在则创建，存在则**追加**内容到文件尾，不会覆盖
    
    ```
    ➜ cat a.txt
    hello
    ➜ echo "world" >> a.txt
    ➜ cat a.txt
    hello
    world
    ```
    
### " < "

- 格式`{command} < filename`

- 将文件的内容作为标准输入，传给左边的命令处理

- 将字符`o`替换为`x`，但是源文件并不会改动

    ```
    ➜ tr 'o' 'x' < a.txt
    hellx
    wxrld
    ➜ cat a.txt 
    hello
    world
    ```

- 或者之前博客提到的用`nc`来传输文件

    ```
    nc -l {port} > filename // 接收端
    nc {接受者IP} {port} < filename // 发送端
    ```
    发送端将文件作为标准输入传输给接受端，再由接收端写入文件

---

## 管道

> 管道命令操作符：**|** ，它能将前面一个命令传出的标准输出(**stdout**)，作为标准输入(**stdin**)传给下一条命令。相比于重定向，管道符号两边都是**可独立执行**的命令，通常用于命令间**传递参数**，而重定位符号的右边必须是**文件名**

- 流程图
![pipe](/1.png)

- 先来看这条命令

    ```
    ➜ ls | grep "**.png"
    arguments.png
    mvvm.png
    nrm-test.png
    require.png
    ```
    `ls`列出当前目录下所有非隐藏文件，作为标准输入传给`grep`去过滤出以`.png`结尾的文件
   
- 现在我用**IDEA**启动了一个**Spring Boot**项目

    ```
    ➜ ps -ef | grep java
      501  6292  6030   0 11:00PM ttys000    0:11.42 /usr/bin/java -jar /Users/s1mple/Library/Mobile Documents/com~apple~CloudDocs/Works/testvue/target/testvue-0.0.1-SNAPSHOT.jar
      501  6324  6224   0 11:05PM ttys001    0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn java
    ```
    执行这条命令后终端打印出**Java**进程信息，进程号**6292**。下面一个是**grep**进程，无关紧要
    
- 接下来我想结合管道命令`kill`掉这个**Java**进程

    ```
    ➜ ps -ef | grep java | grep -v grep | awk '{print $2}' | xargs kill -9   
    ➜ ps -ef | grep java
      501  6352  6224   0 11:11PM ttys001    0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn java
    ```
    只剩下**grep**进程了，**IDEA**控制台也显示 **"Process finished with exit code 137 (interrupted by signal 9: SIGKILL)"** ，该进程已经被`kill`了，其实相当于执行了`kill -9 6292`
    
- 这个命令看上去很长但是不要害怕，它无非是多个命令组合在一起。我之所以想说这个是因为在**Tomcat**的运行脚本`catalina.sh`中看到了用于停止**Tomcat**的类似的命令
    
    ```
    ➜ ps -ef | grep java | grep -v grep | awk '{print $2}' | xargs kill -9 
    ```
    `ps -ef`列出所有进程传递给`grep java`过滤出包含**java**关键字的进程，再传递给`grep -v grep`剔除包含**grep**关键字的进程，`awk '{print $2}'`扫描输入获取第二个参数即进程号**6292**，再作为**xargs**参数传递给`xargs kill -9`最终杀掉进程
    
- 很多时候比如说再写**shell**脚本时确实这样用，因为进程号是动态分配的，不能写死，只能通过一些手段动态获取进程号再进行**参数传递**
    
---
## 命令替换

> 另外简单提一下**命令替换**。在Linux命令行模式下，当遇到一对 **\`\`** 时，将首先执行 **\`** 中间包含的命令，然后将其输出结果作为参数代入命令行中，这就是命令替换了。它**将一个命令的输出作为另外一个命令的参数**
 
- 过滤出`.*rc`的文件，将文件中的内容统一输出到`rc.txt`中

    ```
    ➜ cat `ls -a | grep ".*rc"` > ./Desktop/rc.txt
    ```

- 上诉命令和下面这条起一个作用

    ```
    ➜ ls -a | grep ".*rc" | xargs cat > ./Desktop/rc.txt
    ```

---

## References

- [Streams, pipes, and redirects](https://www.ibm.com/developerworks/linux/library/l-lpic1-103-4/index.html)
    