---
title: Vue 初识
date: 2017-12-14T11:40:31.000Z
tags: ['Vue.js', 'web']
categories: [Coding]
---
## What is Vue.js

- 先来看一下[官网的解释](https://cn.vuejs.org/v2/guide/index.html)
> Vue (读音 /vjuː/，类似于 view) 是一套用于构建用户界面的**渐进式框架**。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库**只关注视图层**，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与现代化的工具链以及各种支持类库结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

- 相比之下，我更喜欢下面这段介绍
> Vue.js 是一个用于构建交互式 Web 界面的 JavaScript 开发库，通过简洁的 API 提供高效的**数据绑定**机制和灵活的**组件系统**。类似于 AngularJS 等优秀前端框架，Vue.js 提供了 MVVM 数据绑定以及一个可组合的组件系统。从技术上讲，Vue.js 着眼于在 MVVM 模式中的**视图模型层**（VM），通过双向数据绑定连接视图和模型，实际的 DOM 操作和输出格式被抽象为**指令**和**过滤器**。

- 上面提到了Vue的几个特性
    - **数据驱动视图**
    - **指令(Directives)**
    - **过滤器(Filter)**
    - **组件化**

---

## Hellovue.html

- 我们先来看一个最简单的**Vue**的**Helloworld**程序，这只是个HTML文件，只不过导入了**Vue**的包。页面显示的内容是 **"Hello Vue!"**

    ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <script src="https://unpkg.com/vue"></script>
    </head>
    <body>
        <div id="app">
            {{ msg }}
        </div>
        <script>
            var app = new Vue({
              el: '#app',
              data: {
                msg: 'Hello Vue!'
              }
            })
        </script>
    </body>
    </html>
    ```
    
- **el**全称**element**，提供一个在页面上已存在的**DOM**元素作为**Vue实例**的**挂载**目标。可以是**CSS选择器**，也可以是一个**HTMLElement实例**

- `el: '#app'`的意思是: 将new出来的Vue实例挂载到`id="app"`的DOM元素上

- 正是因为挂载，`{{ msg }}`才会被data中的msg替换

#### Tips

- [CSS选择器参考手册](http://www.w3school.com.cn/cssref/css_selectors.asp)

    ```
    el : ‘body’ ==> Html element
    el : ‘#app’ ==> id=‘app’
    el : ‘.app’ ==> class=‘app’
    ```
    
- 在浏览器的控制台中可以这样访问
    
    ![Screen-Shot-2017-12-17-at-5.57.26-PM](/0.png)
   
---

## Directives

> 带有前缀 **"v-"** 的是Vue提供的特有的**指令(Directives)**。它们会在渲染的 DOM 上应用特殊的响应式行为。Vue中有许多这样的指令，提供了更简单的编程实现

- 先来看个例子，修改下面输入框的内容，查看效果
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <script src="https://unpkg.com/vue"></script>
    </head>
    <body>
        <div id="app">
            <input type="text" placeholder="在这里输入文字，下面会跟着变化" v-model="msg">
            <p>{{ msg }}</p>
        </div>
        <script type="text/javascript">
            var app = new Vue({
              el: '#app',
              data: {
                msg: 'Hello Vue!'
              }
            })
        </script>
    </body>
    </html>

- 要实现上面的功能，传统的写法是`<input>`标签中加一个事件监听，比如说`onkeyup="change()"`，然后在`<script>`写这个change方法

- 然而Vue中实现只需要加个指令`v-model="msg"`，它的意思是将数据和视图**双向绑定**，只要其中一个改变必定引起另一个的改变，达到了**数据驱动视图**的效果

    ```
    <body>
        <div id="app">
            <input type="text" placeholder="在这里输入文字，下面会跟着变化" v-model="msg">
            <p>{{ msg }}</p>
        </div>
        <script>
            var app = new Vue({
              el: '#app',
              data: {
                msg: 'Hello Vue!'
              }
            })
        </script>
    </body>
    ```

---

## Filter

> **过滤器(Filter)** 可被用于一些常见的文本格式化。过滤器可以用在两个地方：**双花括号插值**和**v-bind表达式**中，由管道 "**|**" 符号指示

- 下面的代码是我基于上一个实例加了转化为大写的过滤器
    
    ```
    <body>
        <div id="app">
            <input type="text" placeholder="在这里输入文字，下面会跟着变化" v-model="msg">
            <p>{{ msg | uppercase }}</p>
        </div>
        <script>
            Vue.filter('uppercase',function(value){
                return value.toUpperCase()
            })

            var app = new Vue({
              el: '#app',
              data: {
                msg: 'Hello Vue!'
              }
            })
        </script>
    </body>
    ```

---

## References

- [官方中文文档](https://cn.vuejs.org/v2/guide/index.html)
- [简明Vue.js教程](https://www.devbean.net/2016/05/brief-vue-js-1/)
- [Vue的特性精华](http://www.jianshu.com/p/2d5f99076c17)
