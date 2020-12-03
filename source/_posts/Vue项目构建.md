---
title: Vue 项目构建
date: 2017-12-13T06:36:50.000Z
tags: ['Vue.js', 'Web']
categories: [Coding]
---
## Creating

> **vue-cli**作为一款mvvm框架语言(vue)的脚手架，集成了**webpack**环境及主要依赖，对于项目的搭建、打包、维护管理等都非常方便快捷。

- 安装**node.js**。安装完成会自动设置环境变量

    ```
    brew install node    // Mac 
    apt-get install node    // Ubuntu
    ```
    现在可以使用**node**及**npm**命令
    ```
    ➜ node -v ; npm -v
	v8.9.2
	5.5.1
    ```
    
    #### Tips
    
    > **Node.js**之于JavaScript类似于JVM之于Java，它是JavaScript的运行平台，使之脱离浏览器得以运行。传统意义上的JavaScript运行在浏览器上，这是因为**浏览器内核**实际上分为两个部分:渲染引擎和JavaScript引擎。前者负责渲染HTML + CSS，后者则负责运行JavaScript。Chrome使用的JavaScript引擎是V8。Node.js是一个运行在服务端的框架，它的底层就使用了V8引擎。
    
- 建议将npm源更改为国内淘宝的镜像，npm全称为**Node Package Manager**

    ```
    npm config set registry https://registry.npm.taobao.org
    ```
    可以执行`npm config list`查看npm配置
    
- 安装**vue-cli**，`-g`代表**global**全局安装，否则只在当前目录下安装

    ```
    npm install -g vue-cli
    ```

- 初始化一个项目，`vue init webpack projectname`
![Screen-Shot-2017-12-13-at-2.28.39-PM-1](/0.png)

    #### Tips

    > **vue-router**是路由组件，后面构建项目会用到。
    > **Eslint**是代码规范的插件，不规范的代码会报错，有助于项目成员统一编码风格。
    > 后面两个都是测试相关，暂时不用。
    
- 初始化成功后会提示你安装依赖以及如何运行

    ```
    To get started:
    
      cd vuedemo
      npm install
      npm run dev
    ```
    
## Running

- 不着急安装依赖，我们先用**WebStorm**打开项目
![Screen-Shot-2017-12-13-at-3.29.38-PM-1](/1.png)
    其中，`.babelrc`是**babel**配置文件，可以将ES6代码转为ES5代码。`.editorconfig`用于统一团队中的编码风格，比如缩进规则等
    
- `package.json`包含了项目的信息和所需的依赖，定义了三个脚本`dev|start|build`，可以在IDE中点击运行。`devDependencies`表示只有开发环境需要的依赖，比如Babel这种开发时的转码工具，生产环境即浏览器并不需要。`^2.5.2`表示向后兼容，实际下载的版本一定是大于等于该版本的

    ```json
    {
      "name": "vuedemo",
      "version": "1.0.0",
      "description": "A Vue.js project",
      "author": "s1mple <798342418@qq.com>",
      "private": true,
      "scripts": {
        "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
        "start": "npm run dev",
        "build": "node build/build.js"
      },
      "dependencies": {
        "vue": "^2.5.2",
        "vue-router": "^3.0.1"
      },
      "devDependencies": {
        "autoprefixer": "^7.1.2",
        "babel-core": "^6.22.1",
        ...
    ```

    #### Tips

    - **version** Must match version exactly
    - **>version** Must be greater than version
    - **>=version** etc
    - **<version**
    - **<=version**
    - **~version** "Approximately equivalent to version"
    - **^version** "Compatible with version"
    - **1.2.x** 1.2.0, 1.2.1, etc., but not 1.3.0
    
- 在执行`npm install`命令后npm根据`package.json`拉取依赖。并在当前目录下生成**node_modules**和**package-lock.json**，前者保存所需依赖库文件，后者则含有依赖的**具体版本**信息，这样做是为了保证每次build拉取的依赖包版本一致，所以命名为**lock**

- 执行`npm run dev`或者在IDE中点击**start**即可启动。会提示编译成功并且访问Url为 http://localhost:8080

- 在IDE中执行`start`时其实执行了`npm run dev`，而`dev`又执行了`webpack-dev-server`
![Screen-Shot-2017-12-13-at-10.27.43-PM](/2.png)

## What's webpack

> 本质上，**webpack**是一个现代 JavaScript 应用程序的模块打包器(**module bundler**)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个**bundle**。

- 如果我们的js代码中`require|import`了一个js依赖包，**webpack**可以方便的将我们js打包成一个**bundle**包，而不是像传统的HTML中`<script>`去引用一堆js包
![webpack](/3.png)

- 在执行`build`时会去读取**webpack**的相关配置文件，`webpack.*.config.js`中定义了程序入口**entry**，输出的文件路径和文件名等
    
    ```js
    module.exports = {
      context: path.resolve(__dirname, '../'),
      entry: {
        app: './src/main.js'
      },
      output: {
        path: config.build.assetsRoot,
        filename: '[name].js',
    ```

- IDE中点击**build**或执行`node build/build.js`会生成编译好的静态文件**dist**，目录如下

    ```
    dist
    ├── index.html
    └── static
        ├── css
        └── js

    ```

## References

- [Vue.js官方中文文档](https://cn.vuejs.org/)
- [webpack官方中文文档](https://doc.webpack-china.org/concepts/)
- [简书--vue学习路径](http://www.jianshu.com/p/7f21880af897?utm_campaign=maleskine&utm_content=note&utm_medium=pc_all_hots&utm_source=recommendation)
- [腾讯云论坛文档--浅入浅出Vue.js](https://www.qcloud.com/community/article/430630001490779316)