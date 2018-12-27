---
title: JavaScript 相关介绍
date: 2017-12-26T12:37:23.000Z
tags: ['JavaScript', 'Node.js']
categories: [Languages]
---
## Preface

> **JavaScript**是目前前端开发中最流行的语言，在学习的过程中会遇到各种**JS**有关的名词。本文章的主要目的是介绍**ECMAScript**(or **ES**)、**CommonJS**、**Node.js**、**Vue.js**或**AngularJS**等与**JavaScript**之间的联系

---

## ECMAScript

> **ECMAScript**和**JavaScript**的关系是，前者是后者的**规范**，后者是前者的一种**实现**（另外的实现还有**Jscript** 和**ActionScript**等）
 
- **JavaScript**的创造者**Netscape**公司，将**JavaScript**提交给国际标准化组织**ECMA**，希望这种语言能够成为国际标准，标准委员会最终决定，标准在每年的6月份正式发布一次，作为当年的正式版本

- 目前的几乎所有的浏览器（不包括低版本的IE），都已经支持**ES5**了，而写代码时最好采用**ES6**语法（毕竟语法糖写起来更舒服，还添加了很多必要的新特性，而且浏览器全面支持**ES6**是迟早的事）。不用担心浏览器不能识别新语法，**Bable**就是用来解决这种问题的工具

- **Babel**是一个广泛使用的**ES6**转码器，可以将**ES6**代码转为**ES5**代码，从而在浏览器环境运行

    ```
    // 转码前
    input.map(item => item + 1);

    // 转码后
    input.map(function (item) {
      return item + 1;
    });
    ```

---

## Node.js

> 在介绍**CommonJS**之前有必要先介绍一下**Node.js**

- 首先明确一点，**JavaScript**和**Python**一样，是一种脚本语言，或者是**解释型语言**，和**C、C++** 这种需要编译成可执行文件再运行的语言不同，它的运行只需要解释器能够解释执行即可，而不需要编译器。**浏览器**就包含了解释执行网页中的**JavaScript**代码的功能

- 伴随着互联网和**JavaScript**语言的快速发展，人们不仅仅局限于拿JS来做前端开发，所以**Node.js**随之应运而生

- **Node.js**作为一个后端的**Javascrip**t运行环境（解释器），这意味着你可以编写系统级或者服务器端的**Javascript**代码，交给**Node.js**来解释执行

- **Node.js**采用了**Google Chrome**浏览器的V8引擎（**C++编写**），性能很好，同时还提供了很多系统级的API，如文件操作、网络编程等。浏览器端的**Javascript**代码在运行时会受到各种安全性的限制，对客户系统的操作有限。相比之下，**Node.js**则是一个全面的后台运行时，基于**Javascript**提供了其他语言能够实现的许多功能

---

## CommonJS

> 可以说**CommonJS**和**Node.js**的关系是，前者是后者的**规范**，后者是前者的一种**实现**

- **ECMAScript**提供核心语言功能，浏览器端的**JavaScript实现**包括了**ES核心**、**DOM**(or **Document Object Model**)和**BOM**(or **Browser Object Model**)，所以你在浏览器的**JS**控制台可以使用`document、window、alert...`等有关**DOM**和**BOM**的操作

- 而**CommonJS**主要为**非浏览器的应用**定义一套API（比如**Node**应用），从而填补了空白。它的终极目标是提供一个类似**Python、Ruby**和**Java**的**标准库**

- 可以看到在**Node**环境中没有定义`alert`和`document`，因为**Node**不关心**DOM**和**BOM**操作

    ```
    ➜ node
    > alert('!!!')
    ReferenceError: alert is not defined
    > document
    ReferenceError: document is not defined
    ```

- 但是它可以调用读取文件的函数，读取出了`a.txt`中的`Hello!`并打印，这是浏览器做不到也不应该去做的

    ```
    ➜ node
    > const fs = require('fs');                 
    > const content = fs.readFileSync('./a.txt','utf-8');
    > console.log(content);
    hello!
    ```

---

## Vue.js、AngularJS...

> 这些没什么好说的，之所以带上**JS**就是因为它们是用**JavaScript**语言开发的框架，实际上就是个**JS库**，用于简化我们前端代码的编写

- 至于为什么使用**Vue.js**这些框架需要安装**Node.js**，原因也很简单，因为很多工具（比如**Babel、webpack**）都是用**Node**开发出来的，这也是你为什么可以用`npm`安装的原因

---

## Reference

- [ECMAScript6简介 - 阮一峰](http://es6.ruanyifeng.com/#docs/intro)
- [CommonJS官网](http://www.commonjs.org/)
- [Node.js官网](https://nodejs.org/en/)
