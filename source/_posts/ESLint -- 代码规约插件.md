---
title: ESLint —— JavaScript代码规约插件
date: 2018-04-04T02:47:12.000Z
tags: [JavaScript]
categories: [Languages]
---
## 前言

ESLint 是一款 JavaScript 的代码规约插件，可以自定义规则，不符合规则的代码可以设置不同级别的报错。方便对于团队内成员的代码质量进行统一管理。之前在 Vue 项目中使用`vue-cli`时可以自动添加 ESLint 代码规约插件，所以也没去管怎么手动添加 ESLint。但是最近在`Koa2`项目中，需要手动的去添加并配置 ESLint。本文将介绍如何安装以及配置 ESLint 的规则。

## 安装

在项目目录下使用`npm`命令安装 ESLint，并且是开发时依赖需要加上`--save-dev`。不推荐全局安装，这样可以实现项目规则的**自定义化**以及团队成员的**统一化**。

```sh
npm install --save-dev eslint
```

初始化，会询问一些问题用于设置 ESLint，比如回答使用 Airbnb 的风格，不同的风格会有一些预定义的规则。

```sh
➜ ./node_modules/.bin/eslint --init

? How would you like to configure ESLint? Use a popular style guide
? Which style guide do you want to follow? Airbnb
? Do you use React? No
? What format do you want your config file to be in? JavaScript
```

完成后会在项目目录下生成`.eslintrc.js`文件，除此之外还可以手动添加`.eslintignore`，作用和`.gitignore`一样用于 Eslint 忽略检查的文件。

接着需要在 WebStorm 中启用 Eslint，勾选`Enable`。

![E9C4D048-BFC5-4D13-A4E8-30CD61BEE953](/0.png)

现在代码检测已经启动了，比如规则中定义了句尾不加分号，如果加了分号就会报错。

![8EAC31A2-7E35-4016-9D0F-0FB7AD8057A6](/1.png)

## 自定义规则

配置自定义规则，以下是我的规则。参考官方文档 [Rules](https://eslint.org/docs/rules/)。

```
module.exports = {
  extends: 'airbnb-base',
  // add your custom rules here
  rules: {
    // allow debugger during development
    'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    // allow console during development
    'no-console': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    // allow unused variables during development
    'no-unused-vars': process.env.NODE_ENV === 'production' ? 'error' : 'warn',
    // disallow trailing commas
    'comma-dangle': ['error', 'never'],
    // disallow trailing semi
    'semi': ['error', 'never'],
    // allow the unary operators ++ and --
    'no-plusplus': 'off',
    // enforce a maximum line length
    'max-len': [0, 150]
  }
}
```

## 运行

在`package.json`中添加命令脚本。

```json
"scripts": {
    // ...
    "lint": "eslint --ext .js ."
}
```

这样就可以使用`npm run lint`运行 Eslint 检查程序，结果列出了问题文件所在路径、警告的级别及问题原因。

```
➜ npm run lint

> airbi-metadata@0.1.0 lint /Users/s1mple/Desktop/airbi-metadata
> eslint --ext .js .



    /Users/s1mple/Desktop/airbi-metadata/app.js
       3:22  error  Extra semicolon                                                 semi
      22:31  error  Use path.join() or path.resolve() instead of + to create paths  no-path-concat
      22:31  error  Unexpected string concatenation                                 prefer-template
      24:15  error  Use path.join() or path.resolve() instead of + to create paths  no-path-concat
      24:15  error  Unexpected string concatenation                                 prefer-template
    
    /Users/s1mple/Desktop/airbi-metadata/routes/index.js
       3:29  warning  'next' is defined but never used  no-unused-vars
       9:35  warning  'next' is defined but never used  no-unused-vars
      13:33  warning  'next' is defined but never used  no-unused-vars
    
    /Users/s1mple/Desktop/airbi-metadata/routes/users.js
      5:17  error    Unexpected function expression    prefer-arrow-callback
      5:32  warning  'next' is defined but never used  no-unused-vars
      9:20  error    Unexpected function expression    prefer-arrow-callback
      9:35  warning  'next' is defined but never used  no-unused-vars

```

## 参考

- [ESLint 官方文档](https://cn.eslint.org/docs/user-guide/getting-started)
- [ESLint Rules](https://eslint.org/docs/rules/)