---
title: 50 行代码实现一个并发的 Python 爬虫程序
date: 2018-12-03T13:41:09.000Z
tags: [Python]
categories: [Projects]
---
## 前言

得益于 Python 丰富的库，我们可以不用重复造轮子而是直接拿人家现成的库用，比如爬虫所需的解析 Html 功能都不用自己亲自写。所以，我在用 Python 改写之前的 Java 爬虫时，只用了 50 行代码就实现了原有功能。本文主要介绍编码时用到的库，以及总结了一些 Python 编码的知识点。

案例还是用的之前 [Java 网络爬虫](https://s2mple.xyz/2018/11/05/Java%20%E7%BD%91%E7%BB%9C%E7%88%AC%E8%99%AB/) 的案例。所需 `Python Version >= 3.6`，用到的库有：

- **beautifulsoup4** 三方库用于解析 Html，执行 `pip install beautifulsoup4` 安装
- 内置的 **urllib** 用于发起网络请求获取响应内容
- 内置的 **concurrent.futures** 并发库中的 **ProcessPoolExecutor** 用于创建进程池

## 源码

源码已经上传到 [GitHub](https://gist.github.com/s1mplecc/dfd15f58cbbe5fad2ab13bc2246d49f4)

```python
import os
import re
import time
from urllib import request
from concurrent.futures import ProcessPoolExecutor
from bs4 import BeautifulSoup

WWW_BIQUGE_CM = 'http://www.biquge.cm'


def __fetch_html(url, decode='UTF-8'):
    req = request.Request(url)
    req.add_header('User-Agent', 'Mozilla/5.0')
    with request.urlopen(req) as res:
        return res.read().decode(decode)


def __parse_title_and_hrefs(index):
    html = __fetch_html(f'{WWW_BIQUGE_CM}/{index}/', 'gbk')
    soup = BeautifulSoup(html, 'html.parser')
    links = soup.find_all('a', href=re.compile(rf'/{index}/'))
    hrefs = list(map(lambda x: x['href'], links))
    title = soup.h1.string
    print(f'title: {title}\nhrefs: size={len(hrefs)}')
    return title, hrefs


def __fetch_content(href):
    print(f'parsing {href}')
    html = __fetch_html(f'{WWW_BIQUGE_CM}/{href}', 'gbk')
    soup = BeautifulSoup(html, 'html.parser')
    return re.sub(r'<div id="content">|</div>|<br/>', '', str(soup.find('div', id='content')))


def __append_contents_to_file(title, hrefs):
    with ProcessPoolExecutor(16) as executor:
        contents = executor.map(__fetch_content, hrefs)
    os.makedirs('downloads', exist_ok=True)
    with open(f'./downloads/{title}.txt', 'wt+') as f:
        for content in contents:
            f.write(content)


def run(index='12/12481'):
    start = time.time()
    title, hrefs = __parse_title_and_hrefs(index)
    __append_contents_to_file(title, hrefs)
    print(f'spend time: {time.time() - start}s')


if __name__ == '__main__':
    run('9/9422')

```

### urllib

模拟浏览器发起一个 HTTP 请求，我们需要用到 `urllib.request` 模块。其 `urlopen()` 方法用于发起请求并获取响应结果，该方法可单独传入一个 `urllib.request.Request` 对象，并返回一个 `http.client.HTTPResponse` 对象。

```python
def __fetch_html(url, decode='UTF-8'):
    req = request.Request(url)
    req.add_header('User-Agent', 'Mozilla/5.0')
    with request.urlopen(req) as res:
        return res.read().decode(decode)
```

使用 `Request` 包装请求头。如果不设置 headers 中的 **User-Agent**，默认的 User-Agent 是 Python-urllib。可能一些网站会将该请求拦截，所以需要伪装成浏览器发起请求。

### Beautiful Soup

> Beautiful Soup is a Python library for pulling data out of HTML and XML files. It works with your favorite parser to provide idiomatic ways of navigating, searching, and modifying the parse tree. It commonly saves programmers hours or days of work.

Beautiful Soup 用于从 HTML 或 XML 文件中提取数据，提供的功能非常强大。它支持 Python 标准库中的 HTML 解析器：`BeautifulSoup(markup, "html.parser")`，还支持一些第三方的解析器（如 html5lib、lxml 等）。

Beautiful Soup 将 HTML 文档转换成一个复杂的树形结构,每个节点都是 Python 对象,所有对象可以归纳为4种: `Tag`，`NavigableString`，`BeautifulSoup`，`Comment`。`Tag` 对象与 XML 或 HTML 原生文档中的 tag 相同，所以非常适合用于**定位**。下面列出一些常见用法，感兴趣的同学可以查阅 [官方文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/)。

```python
soup.title
# <title>The Dormouse's story</title>

soup.a
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.find('a', id='link3')
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>

for link in soup.find_all('a'):
    print(link.get('href'))
    # http://example.com/elsie
    # http://example.com/lacie
    # http://example.com/tillie
```

### 并发

从 Python3.2 开始 `concurrent.futures` 被纳入了标准库，这个模块中有2个类：`ThreadPoolExecutor` 和 `ProcessPoolExecutor`，分别对 `threading` 多线程库和 `multiprocessing` 多进程库的进行了高级别的抽象，封装了统一的接口。

关于是使用多线程还是多进程，大部分人可能有所耳闻，Python 推荐使用多进程而不是多线程。我自己测试的情况也是爬取 3000 章时使用 `ProcessPoolExecutor` 大约需要 20s，使用 `ThreadPoolExecutor` 需要大概 40s，性能差了一倍。

其他语言，CPU 是多核时是支持多个线程同时执行的。但在 Python 中，无论是单核还是多核，**同时只能由一个线程在执行**，其根源是 **GIL** 的存在（只存在于 CPython，PyPy 和 Jython 中没有）。

GIL 全称 **Global Interpreter Lock**(全局解释器锁)，是 Python 设计之初为了数据安全所做的决定。某个线程想要执行，必须先拿到 GIL，并且在一个 Python 进程中只有一个 GIL。并且每次释放 GIL 锁时线程会进行锁竞争，切换线程也会消耗资源。这就是为什么在多核 CPU 上 Python 的多线程效率并不高的原因所在，以至于 Python 的专家们精心制作了一个标准答案：**不要使用多线程，请使用多进程**。

此外，Python 可使用 **perf** 库进行性能测试，以下是爬取 50 章时的性能：

```sh
(venv) ➜  crawler ✗ python3 -m perf timeit 'import app;app.run("12/12455")'
...
* the standard deviation (281 ms) is 24% of the mean (1.16 sec)
* the maximum (2.02 sec) is 73% greater than the mean (1.16 sec)

Mean +- std dev: 1.16 sec +- 0.28 sec
```

## 其他

### venv

**venv**，全称 **virtualenv** 虚拟环境。用过 JavaScript 的同学都知道，执行 `npm install xxx` 时会在当前目录生成一个 `node_modules` 目录，依赖会被安装在这个目录下，除非你加上 `-g/--global` 全局参数，否则安装的依赖只对当前项目生效（不是全局依赖）。这其实相当于做了一层隔离，将当前项目的环境与全局环境隔离开，有利于版本的管理。

venv 也是这样的作用，用于**为一个应用创建一套隔离的 Python 运行环境**。在这个环境中，你可以管理 Python 版本，pip 版本，以及你所用的三方库的版本，而不会与全局环境冲突。

如果你使用的是 PyCharm，那么创建项目时就可以勾选使用 venv（这也是建议的选择）。效果如下图：
![Screen-Shot-2018-12-03-at-11.01.47-AM](/0.png)命令行多了 (venv) 前缀：
```
(venv) ➜  crawler ✗
```

如果你不是使用的 PyCharm，参考这篇文档：[virtualenv](https://www.kancloud.cn/smilesb101/python3_x/298883)

### 风格规范

Python 风格规范我参考的 [Google 开源项目风格指南](https://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/python_style_rules/#id16)。推荐使用 **PyCharm** IDE，和 IntelliJ IDEA 一样产自 JetBrains 公司，非常良心的软件，会有一些风格提示，并且可以使用快捷键（Ctrl/Cmd + Shift + L）自动格式化。这里主要说说命名吧。

#### 命名

**应该避免的名称**
1. 单字符名称, 除了计数器和迭代器
2. 包/模块名中的连字符(-)
3. 双下划线开头并结尾的名称( Python 保留, 例如`__init__`)

**命名约定**
1. 用**单下划线(_)开头**表示模块变量或函数是 protected 的(使用 `from module import *` 时不会包含)。
2. 用**双下划线(__)开头**的实例变量或方法表示类内私有。
3. 对类名使用大写字母开头的单词(如 CapWords，即 Pascal 风格)，但是模块名应该用小写加下划线的方式(如 lower_with_under.py )。尽管已经有很多现存的模块使用类似于CapWords.py 这样的命名，但现在已经不鼓励这样做，因为如果模块名碰巧和类名一致，这会让人困扰。

**Python之父Guido推荐的规范**

Type	|Public|	Internal
----|----|------
Modules	|lower_with_under	|_lower_with_under
Packages|	lower_with_under	 |
Classes	|CapWords|	_CapWords
Exceptions	|CapWords	 |
Functions	|lower_with_under()|	_lower_with_under()
Global/Class Constants	|CAPS_WITH_UNDER	|_CAPS_WITH_UNDER
Global/Class Variables	|lower_with_under	|_lower_with_under
Instance Variables	|lower_with_under	|_lower_with_under (protected) or __lower_with_under (private)
Method Names	|lower_with_under()|	_lower_with_under() (protected) or __lower_with_under() (private)
Function/Method Parameters|	lower_with_under	 |
Local Variables|	lower_with_under	 |

#### Main

**所有的顶级代码在模块导入时都会被执行**。即使是一个打算被用作脚本的文件，也应该是可导入的。并且简单的导入不应该导致这个脚本的主功能(main functionality)被执行，这是一种副作用。主功能应该放在一个函数中，并在执行主程序前总是检查 `if __name__ == '__main__'`。

```python
def main():
      ...

if __name__ == '__main__':
    main()
```

`__name__` 是内置变量，指代**当前模块名**，当模块被直接运行时模块名为 `__main__`。这句话的意思就是，当模块被直接运行时，下面代码块将被运行，当模块是被导入时，代码块不被运行。

### 文件写入

```python
os.makedirs('downloads', exist_ok=True)
with open(f'./downloads/{title}.txt', 'wt+') as f:
    for content in contents:
        f.write(content)
```

第一句用于创建 downloads 文件夹，`exist_ok=True` 参数允许创建的文件夹已存在，否则会抛出 `FileExistsError` 异常。

第二句 `with ... as ...` 的用法和 Java 中的 `try with resources` 有点类似，这里它会自动关闭打开的文件流。它的核心思想是 with 所求值的对象必须有一个 `__enter()__` 方法和一个 `__exit()__` 方法，在 with 代码块开始和结束这两个方法会被执行。如果出现异常则会执行 `__exit()__`，并传入三个参数 `exc_type`，`exc_value`，`exc_traceback` 用于异常处理。

`open('abc.txt', 'a+')` 打开文件时不同的参数有不同的作用，我这里用的 `wt+` 表示以文本格式打开一个文件用于读写，如果该文件已存在则将其覆盖，如果该文件不存在则创建新文件。还有其他参数，例如 `a+` 代表 append 追加，`r` 表示只读等等。

## 参考

- [廖雪峰的Python3.x教程](https://www.kancloud.cn/smilesb101/python3_x/295557)
- [Python 风格指南 —— Google 开源项目风格指南](https://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/python_style_rules/#id16)
- [详解 python3 urllib](https://www.jianshu.com/p/2e190438bd9c)
- [Beautiful Soup 4 官方文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/)
- [concurrent.futures — Launching parallel tasks 官方文档](https://docs.python.org/3/library/concurrent.futures.html)
- [concurrent.futures 性能分析](https://blog.louie.lu/2017/08/01/%E4%BD%A0%E6%89%80%E4%B8%8D%E7%9F%A5%E9%81%93%E7%9A%84-python-%E6%A8%99%E6%BA%96%E5%87%BD%E5%BC%8F%E5%BA%AB%E7%94%A8%E6%B3%95-06-concurrent-futures/)