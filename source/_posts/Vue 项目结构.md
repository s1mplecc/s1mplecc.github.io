---
title: Vue 项目结构
date: 2017-12-14T02:11:19.000Z
tags: ['Vue.js', 'Web']
categories: [Coding]
---
## Preface

> 在已经[构建vue-cli项目](https://s1mple.online/vue-cli-project-build/)的基础上，我们来梳理一下vue-cli项目的结构。
 
- vue-cli项目总体结构
![vue-project-structure](/0.png)

## build

- build文件主要是**webpack**打包生成静态文件的配置
![build](/1.png)

- 最新的**vue-webpack-template**中已经去掉了**dev-server.js**改用**webpack-dev-server**代替，**webpack-dev-server**是一个独立的NPM包，支持**热重载**

    ```
     -    "dev": "node build/dev-server.js",
     +    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    ```

## config

- config文件主要是项目相关配置，我们常用的就是当端口冲突时配置监听端口，打包输出路径及命名等
![config](/2.png)

## src

- 这是项目最初的一个结构图。**assets**不用多说，静态资源文件包括css文件、外部的一些js包都可以放这里。**components**和**router**则是Vue的俩个核心概念：**组件**和**路由**

    ![Screen-Shot-2017-12-16-at-5.13.01-PM](/3.png)
    
### main.js

- 项目的入口文件，new了一个Vue实例，将这个实例挂载到**id=app**的**DOM元素**上，并用**App.vue**中的模版替换该元素，这也是为什么我们能看到**App.vue**中的logo图片

    ```
    import App from './App'
    new Vue({
      el: '#app',
      router,
      template: '<App/>',
      components: { App }
    })
    ```
    
### App.vue

- 根组件，可以看到 **.vue**文件包含3个部分：**template、script、style**，分别对应**html、JavaScript、css**文件

    ```
    <template>
      <div id="app">
        <img src="./assets/logo.png">
        <router-view/>
      </div>
    </template>

    <script>
    export default {
      name: 'app'
    }
    </script>

    <style>
    #app {...}
    </style>
    ```
    
- `<router-view>`是路由子视图，类似于一个插槽

### router/index.js

- 路由文件，如果访问跟路径`/`，那么将**HelloWorld**组件插入到`<router-view>`中并渲染出来

    ```
    import Vue from 'vue'
    import Router from 'vue-router'
    import HelloWorld from '@/components/HelloWorld'

    Vue.use(Router)

    export default new Router({
      routes: [
        {
          path: '/',
          name: 'HelloWorld',
          component: HelloWorld
        }
      ]
    })
    ```

## Postscript
    
- 可以看到我在浏览器控制台输入app，看到的是**id=app**的DOM元素，也就是**App.vue**中插入了**HelloWorld.vue**的模版文件

    ![Screen-Shot-2017-12-16-at-5.45.45-PM](/4.png)

## References

- [vue-cli入门——项目结构](http://www.jianshu.com/p/7006a663fb9f)
