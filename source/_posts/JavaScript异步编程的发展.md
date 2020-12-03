---
title: JavaScript 异步编程
date: 2018-04-27T02:47:39.000Z
tags: ['JavaScript']
categories: [Languages]
---
## 前言

你可能知道，JavaScript 语言的执行环境是**单线程（single thread）**。这种模式的好处是实现起来比较简单，执行环境相对单纯；坏处是只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。常见的浏览器无响应（假死），往往就是因为某一段 JavaScript 代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。为了解决这个问题，JavaScript 语言将任务的执行模式分成两种：同步（Synchronous）和 异步（Asynchronous）。

- **同步模式**：后一个任务等待前一个任务结束，然后再执行，程序的执行顺序与任务的排列顺序是一致的、同步的；
- **异步模式**则完全不同，每一个任务有一个或多个回调函数（callback），前一个任务结束后，不是执行后一个任务，而是执行回调函数，后一个任务则是不等前一个任务结束就执行，所以程序的执行顺序与任务的排列顺序是不一致的、异步的。

异步模式非常重要。在浏览器端，耗时很长的操作都应该异步执行，避免浏览器失去响应，最好的例子就是 Ajax 操作。在服务器端，Node.js 的异步 I/O 则保证了同一时间可以响应大量的 Http 请求。

本文主要介绍 JavaScript 的异步编程发展：从回调函数到 ES6 的 Promise 对象，再到 ES7 的 async/await 关键字。示例代码已上传至[GitHub](https://github.com/s1mplecc/js-asynchronous-programming)。

## 回调函数

回调（Callback）函数是异步编程最基本的方法。回调就好比你送女朋友到车站，并让她回家了给你回条短信。回调函数在完成任务后就会被调用，Node.js 就使用了大量的回调函数

- 看看下面的示例代码，你要求女朋友回到家后发一条短信告诉你，然后她在回家的路上愉快的耍起了手机，哈哈哈

```js
function goHome(callback) {
  setTimeout(() => {
    callback()
  }, 2000)
  console.log('I am playing a mobile phone')
}

function sendMessage() {
  console.log('I have just arrived home.')
}

goHome(sendMessage)

// result:
// I am playing a mobile phone.
// I have just arrived home.    // after 2000ms
```
    
注意打印结果的顺序。先打印的是`I am playing a mobile phone`，而后过了 2s 打印`I have just arrived home`。这是因为使用了`setTimeout()`这个函数，表示延时 2000ms 后执行回调函数。

在这个示例中，回调函数即是`sendMessage()`，回家这个动作是比较耗时的，所以我们使用了**异步**的方式，使得**玩手机的行为不受到阻塞**。是吧，玩手机不一定要回家才能完，路上也能玩嘛。回调使得同步操作变成了异步操作，相当于先执行程序的主要逻辑，将耗时的操作推迟执行。

回调函数的优点是简单、容易理解，缺点是不利于代码的阅读和维护，各个部分之间**高度耦合**，流程会很混乱，而且每个任务只能指定一个回调函数。在 Node.js 的开发中，由于逻辑分层所致，会出现多层回调，易于引发异常处理混乱、闭包过于复杂、代码难以维护等问题。

## Promise

Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件，更合理和更强大。它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了 Promise 对象。所谓 Promise，简单说就是一个**容器**，里面**保存着某个未来才会结束的事件（通常是一个异步操作）的结果**。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理。

Promise 对象有以下两个特点：

- 对象的状态不受外界影响。Promise 对象代表一个异步操作，有三种状态：**Pending**（进行中）、**Resolved**（已成功，又称 Fulfilled）和 **Rejected**（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是 Promise 这个名字的由来，它的英语意思就是「承诺」，表示其他手段无法改变。

- **一旦状态改变，就不会再变**，任何时候都可以得到这个结果。Promise 对象的状态改变，只有两种可能：从 Pending 变为 Resolved 和从 Pending 变为 Rejected。只要这两种情况发生，状态就**凝固**了，不会再变了，会一直保持这个结果。如果改变已经发生了，你再对 Promise 对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

来看下面这段代码：

```js
const goHome = new Promise((resolve) => {
  setTimeout(()=>{
    resolve('I have just arrived home.')
  },2000)
  console.log('I am playing a mobile phone.')
})

function sendMessage(message) {
  console.log(message)
}

goHome.then(sendMessage)
```

这样写的优点在于，回调函数变成了链式写法，程序的流程可以看得很清楚，`goHome.then(sendMessage)`显然更符合人的思维方式。
    
我们可以在`Promise`中`reject`错误，在用`catch`去捕捉，你可能在 axios 发送异步请求的时候见过这种写法。`catch`方法是`.then(null, rejection)`的别名，用于指定发生错误时的回调函数。

```js
const isEven = new Promise((resolve, reject) => {
  const num = Math.round(Math.random() * 10)
  console.log('num', num)
  num % 2 === 0 ? resolve(`${num} is an even number`) : reject(`${num} is not an even number`)
})

isEven
  .then(result => console.log(result))
  .catch(err => console.log(err))

// result:
// num 6
// 6 is an even number
// num 7
// 7 is not an even number
```
    
有了 Promise 对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。此外，Promise 对象提供统一的接口，使得控制异步操作更加容易。更多请参考[阮老师的教程](http://es6.ruanyifeng.com/#docs/promise)。

## async/await

最后来看一下异步编程的**终极解决方案**:`async/await`。`async`作为 ES7 的一大特性，已经在 Node.js 的框架——[Koa2](https://koa.bootcss.com/) 中被广泛使用。`async`函数是什么？一句话，**async 函数就是 Generator 函数的语法糖**。`async`用于申明一个`function`是异步的，而 `await`用于等待一个异步函数执行完成。
 
使用时需要注意：**async 函数返回一个 Promise 对象**；`await`只能在`async`内部使用，类似于`async function`内部的`then`命令的语法糖。

`await`是个运算符，用于组成表达式，`await`如果在等待一个 Promise 对象，就会阻塞后面的代码，等着 Promise 对象`resolve`，然后将得到`resolve`的值作为 `await` 表达式的运算结果。

用`async/await`改写后的代码如下，看起来就像同步代码那样直观：

```js
const goHome = new Promise((resolve) => {
  setTimeout(() => {
    resolve('I have just arrived home.')
  }, 2000)
  console.log('I am playing a mobile phone.')
})

async function sendMessage() {
  const msg = await goHome
  console.log(msg)
}

sendMessage().then(() => console.log('Play together.'))

// result:
// I am playing a mobile phone.
// I have just arrived home.    // after 2000ms
// Play together.
```

## 总结

从最早的回调函数，到 Promise 对象，再到 Generator 函数，每次都有所改进，但又让人觉得不彻底。**异步编程的语法目标，就是怎样让它更像同步编程**， async 函数就是隧道尽头的亮光，很多人认为它是异步操作的终极解决方案。实际使用起来也更顺手、更容易理解，它必然会被普及，我们应该习惯这种用法。事实上 Koa2 也正是由于此而比拥有复杂的回调嵌套的 Express 更脱颖而出。

## 参考

- [Javascript异步编程的4种方法 - 阮一峰](http://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html)
- [Promise 对象 - 阮一峰ES6教程](http://es6.ruanyifeng.com/#docs/promise)
- [async 函数 - 阮一峰ES6教程](http://es6.ruanyifeng.com/#docs/async)
- [理解 JavaScript 的 async/await](https://segmentfault.com/a/1190000007535316)