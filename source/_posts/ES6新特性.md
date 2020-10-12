---
title: ES6 新特性
date: 2017-12-29T05:51:55.000Z
tags: [JavaScript]
categories: [Languages]
---
## 前言

**ES6**（or **ECMAScript 6**）规定了新一代的 **JavaScript** 语法，其中含有许多必要且实用的新特性。尤其是加入了一些**语法糖**使我们代码更简洁明了。本文所有代码都在浏览器的 **JavaScript** 控制台或者 **Node** 环境下运行过，输出结果在`// result:`注释后。

## Arrows

> **箭头函数**是使用 **=>** 语法缩写的一个函数（相当于**匿名函数**），左边是参数，右边可以是表达式（**expression**）或者声明体（**statement bodies**）。

`=>`右边接表达式：

```js
console.log([1,2,4].map(v => v + 1))
// result: [ 2, 3, 5 ]
```

`=>`右边接声明体：

```js
console.log([1,2,4].forEach(v => {
    if(v > 2){
        console.log(v)
    }
}))
// result: 4
```

箭头函数可以和**不定参数**配合使用，下面代码定义了一个求和方法：

```js
const sum = (...nums) => {
    let sumValue = 0
    nums.forEach(num => sumValue += num)
    console.log(sumValue)
}
sum(1,2,4)
// result: 7
```
    
需要特别注意的是，箭头函数**绑定this**，这是和 **function** 定义是不同的：

```js
let std = {
    birth: 1990,
    getAge: function () {
        const _this = this
        const fn = function () {
            return new Date().getFullYear() _this.birth：
        }
        return fn()
    }
}
std.getAge()
// result: 28
```

传统的写法我需要特地定义个`_this`让它指向当前这个 **std** 对象，而箭头函数绑定了上下文的 **this**，即调用该 **function** 的对象 **std**：

```js
std = {
    birth: 1990,
    getAge: function () {
        var fn = () => new Date().getFullYear() this.birth // this指向std对象：
        return fn()
    }
}
std.getAge()
// result: 28
```

## Let + Const

> ES6 新增了两个绑定了**块作用域**的声明变量的关键字 **let** 和 **const**。

`let`用法和`var`一样，但是限定了作用域。因为在 **JavaScript** 中，`var`定义全局变量，所以能不用`var`的地方就不用：

```js
{
  let a = 10
  var b = 1
}
// result:
// a ReferenceError: a is not defined.
// b 1
```

`const`声明一个只读的常量，并且必须**初始化**。一旦声明，常量的值就不能改变：

```js
const a // Missing initializer in const declaration
const a = 1
a = 2 // Assignment to constant variable.
```

阮一峰老师的**ES6编程风格**中明确指出在`let`和`const`之间，建议优先使用`const`，尤其是在**全局环境**，不应该设置变量，只应设置常量。**const优于let原因如下:**

- **const**可以提醒阅读程序的人，这个变量不应该改变。
- **const**比较符合**函数式编程思想**，运算不改变值，只是新建值，而且这样也有利于将来的分布式运算。
- **JavaScript** 编译器会对**const**进行优化，所以多使用**const**，有利于**提高程序的运行效率**，也就是说**let**和**const**的本质区别，其实是编译器内部的处理不同。**所有的函数都应该设置为常量**。

## Template Strings

> ES6 引入了构造字符串的语法糖：**模版字符串 \`...`**，使用**反引号**表示。它有两个作用，一是表示**多行字符串**，二是**替换为绑定的变量值**。

多行字符串：

```js
console.log(`第一行
第二行`)
// result:
// 第一行
// 第二行
```

在模版字符串中使用`${property}`替换为属性的值：

```js
const name = 'Bob', time = 'today'
console.log(`Hello ${name}, how are you ${time}?`)
// result: Hello Bob, how are you today?
```

## Default + Rest + Spread

> 参数的默认值、不定参数以及参数数组的展开。

函数的参数可以**指定默认值**，调用时如果未指定将使用默认值：

```js
function f(x, y = 12) {
  return x + y
}
// result:
// f(3) -> 15 
// f(3,4) -> 7
```

**不定参数**使用`...`表示，它取代了之前的 **arguments**，适用于更常见的需求。如下例中 **y** 是一个参数数组：

```js
function f(x, ...y) {
  return x * y.length
}
// result:
// f(3,'abc',10) -> 6
// f(3,'abc',10,'xyz') -> 9
```

同时`...`还可以在函数调用时将数组转换成连续的参数：

```js
function f(x, y, z) {
  return x + y + z
}
f(...[1,2,3])
// result: 6
```

## for ... of

> **for ... of** 是 ES6 新加入的遍历循环的操作，而传统的 **for ... in** 循环由于历史遗留问题，它遍历的实际上是对象的**属性名**。

我们用`for ... in`遍历对象的**属性名**，需要用`o[prop]`来获取属性的值：

```js
const o = { name: 'Jack', age: 11}
for (let prop in o) { 
    console.log(`${prop} is ${o[prop]}`)
}
// result:
// name is Jack
// age is 11
```

同样的，**Array** 实际上也是一个对象，它的每个元素的索引被视为一个属性：

```js
const arr = [1,4,9]
arr.name = 'Jack'
// actually arr is an object

/* (3) [1, 4, 9, name: "Jack"]
0: 1
1: 4
2: 9
name: "Jack"
length: 3
__proto__: Array(0)*/
```

我们使用`for ... in`去遍历数组，实际上`for ... in`遍历的是数组的**下标索引**：

```js
for (let index in arr) { 
    console.log(`数组第${index}个是${arr[index]}`)
}
// result:
// 数组第0个是1
// 数组第1个是4
// 数组第2个是9
// 数组第name个是Jack
```

ES6 引入`for ... of`循环，遍历的是集合本身的元素，可以用于遍历 **Array、Set、Map** 等：

```js
for (let value of arr) { 
    console.log(value)
}
// result:
// 1
// 4
// 9
```

可以看到我们的`name: "Jack"`被舍弃了，实际上也不推荐这种向数组添加属性的操作。

## forEach

更好的方式是直接使用`iterable`内置的`forEach`方法，它接收一个函数，每次迭代就自动回调该函数：

```js
arr.forEach((value,index) => console.log(`${index} : ${value}`))
// result:
// 0 : 1
// 1 : 4
// 2 : 9
```

遍历一个 **Map**：

```js
var m = new Map([['Jack', 11], ['Tom', 13], ['John', 10]])
m.forEach((value, key, map) => console.log(`${key} is ${value} years old.`))
// result:
// Jack is 11 years old.
// Tom is 13 years old.
// John is 10 years old.
```

## Destructuring

> ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为**解构(Destructuring)**。

可以利用数组和解构，一次性给多个变量赋值。同时可以很方便的**交换**两个变量值：

```js
let [a, b] = [1, 2]
// result:
// a = 1
// b = 2

// 交换a、b的值
[a, b] = [b, a]
// result:
// a = 2
// b = 1
```

也可以使用嵌套数组进行解构，左右两边的模版应该相同，不对应则变量值为`undefined`：

```js
let [x, [[y], z]] = [1, [[2], 3]]
// result:
// x = 1
// y = 2
// z = 3
```

`...`被解构成一个数组：

```js
let [a,...rest] = [1,2,3,4]
// result:
// a = 1
// rest = [2, 3, 4]
```

解构还可以用于对象，**需要注意:** 数组的解构是按次序排列的，而对象的属性没有次序，所以**变量必须与属性同名**：

```js
let { bar, foo } = { foo: "aaa", bar: "bbb" }
// result:
// foo = "aaa"
// bar = "bbb"
```

当不知道参数传入顺序时，可以将一组参数与变量名对应起来：

```js
function f({x, y, z}) { 
    return x + y - z ：
}
f({z: 3, y: 2, x: 1})
// result: 0
```

非常适合提取 **JSON** 对象中的属性值：

```js
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
}

let { id, status, data } = jsonData
console.log(id, status, data)
// result:
// 42, "OK", [867, 5309]
```

以及加载模块时：

```js
import { getTable, postLoginForm } from 'api'
```


## Classes

> ES6 引入 **class** 关键字，是的原先对象原型(**prototype**)的写法变得更加直观、更像**面向对象编程**的语法，用法类似于 **Java** 中的 **class**。

定义一个 **Point** 类，其中有个构造函数`constructor`，需要注意类中的函数不需要`function`关键字：

```js
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return `(${this.x},${this.y})`;
  }
}

const p = new Point(1,2) // 生成实例
p.toString()
// result: "(1,2)"
```

## Modules

> **模块**的引入是 ES6 的一大重要特性，我们可以将一个大程序拆成几个小文件，需要的时候再导入。ES6 之前是没有模块加载的功能的，社区制定了一些方案，比如 **CommonJS**的 **require、module.exports**。而 ES6 规定了模块加载的标准语法，并且 ES6 可以在**编译时就完成模块加载**，效率要比 **CommonJS** 模块的加载方式高。

下面代码我在`b.js`中导出了一个常量和一个函数，在`a.js`中导入：

```js
// b.js
module.exports = {
    pi: 3.14,
    plusOne: v => v + 1
}

// a.js
const { pi, plusOne } = require('./b')

console.log(pi)
console.log(plusOne(10))

// 执行 node a.js 输出结果:
// 3.14
// 11
```

在 ES6 中规定了模块导入导出的关键字`import、export`，使用方法如下：

```js
// b.js
const pi = 3.14;
const plusOne = v => v + 1;

export { pi, plusOne }

// a.js
import { pi, plusOne } from './b';
```

`import`大括号中变量名，必须与导出时声明的对外接口的**名称相同**。

`export default`命令用于指定模块的**默认输出**，一个模块只能有一个默认输出，因此`export default`只能出现一次，所以`import`导入时可以不加大括号，名称也可以随意取。其实就想当于导出了一个 **default** 变量，将开平方的箭头函数**赋值**给它。

```js
// b.js
export default v => v * v

// a.js
import square from './b'
console.log(square(10))

// result: 100
```

因此我们才可以这样引入第三方库：

```
import _ from 'lodash'
```

需要**特别注意**的是，目前 **Node** 和浏览器还未对 **import、export** 做支持，因为**V8引擎**暂不支持，但是不久的将来肯定会加入该语法。[Node官方文档](https://nodejs.org/dist/latest-v9.x/docs/api/esm.html#esm_unsupported) 也明确说明了不建议 **require** 而推荐使用标准的 **import** 并且等待最新的**V8引擎**的支持。可以使用 **Babel** 进行转码。

![512494CE-D620-4262-B2A2-CC5B351E6A3C](/0.png)

## 参考

- [Babel官网 —— Learn ES2015](https://babeljs.io/learn-es2015/)
- [阮一峰 —— ES6入门](http://es6.ruanyifeng.com/)
- [廖雪峰 —— JavaScript教程](https://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000)