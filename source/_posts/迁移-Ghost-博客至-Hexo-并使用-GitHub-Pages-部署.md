---
title: 迁移 Ghost 博客至 Hexo 并使用 GitHub Pages 部署
date: 2018-12-27 13:45:37
tags: [Hexo, Ghost]
categories: []
---
## 为什么迁移

### Ghost

优点：
	1. 支持在线编辑文章，随时更新随时发布。
	2. 图片上传方便，直接拷贝到正文即可，会自动上传到服务器并生成 Markdown 链接。
	3. 支持多人同时使用，可以用作团队博客。

![dbdd8ad99f065192aa340cf474709b2d.png](0.png)
![fa0a3c8ae59030f8e5ef80a9b04d8c7d.png](1.png)

缺点：
1. 需要在服务器上部署 Ghost。
2. 本地和服务器端文章同步麻烦（这也是我换用 Hexo 的主要原因）。
3. 不支持离线编辑文章（虽说这个可以本地写完了再一次性拷贝到服务器，但这又回到了缺点 2）。
4. Themes 配置麻烦，不够统一。我的 Kaldorei 主题还是去 GitHub 上克隆的源码，修改了源文件后打包上传的。

### Hexo

![6bc65c0ff0fb6d62ea90ce4c35c04ac9.png](2.png)
![8b1c5aa127adbfef220cfb30a6d55e1d.png](3.png)

优点：
1. 文档本地化。在本地编写博客，一条指令即可生成静态网页。（具体体现就是你在 `source/_posts` 下编写）
2. 可部署到 GitHub Pages，不需要额外的服务器。（配合第 1 条超级好用好嘛！）
3. 丰富的插件和主题，配置也统一化了。比如上图我使用的 Next 主题，Next 的相关配置修改 `_config.yml` 即可；Hexo 的相关配置修改主目录的 `_config.yml` 即可。
4. 图片上传还算方便。在与文章同名的文件夹下保存图片即可。这个下面会细说。

## 安装部署

### GitHub 创建仓库

GitHub Pages 分为两类，用户主页和项目主页。我们需要部署的是用户主页，首先需要在 GitHub 上新建一个仓库。仓库名必须为 `${GitHub username}.github.io`。这样才会被识别为 GitHub Pages。用户主页是唯一的，填其他名称只会被当成普通项目。通过 `https://${GitHub username}.github.io` 进行访问。

### 分支管理

GitHub Pages 会自动部署静态网页文件，并**将 master 分支作为部署的默认分支**。为将静态网页和源文件（包含文章、主题等）分离开，强烈建议创建新分支。`git checkout -b hexo` 创建源码分支。你的文档编辑和 Hexo 操作应当在此分支上完成，这样在执行 `hexo generate -deploy` 时就会生成静态文件（public 文件夹），并将整个 public 文件夹 deploy 到 GitHub 的 master 分支上。

```sh
➜  github.io git:(hexo) ✗ gb -v    

* hexo                  0d03653 dump ghost markdown files to hexo
  master                673d037 initial commit
```

### 安装 Hexo

Hexo 和 Ghost 一样，都是使用 Node.js 编写的，所以使用 npm 安装即可。

```sh
npm install hexo-cli -g
```

### 生成 Hexo 项目

现在你可以使用 `hexo` 命令了，为了生成 Hexo 项目首先需要确保一件事：你已将 Git 仓库 clone 至本地，并且处于 hexo 分支上。由于必须在空文件夹下生成 Hexo 项目，所以我们先将 `.git` 文件夹整个移出去，然后执行 `hexo init` 命令。最后别忘了将 `.git` 文件夹移回来。

```sh
➜  github.io git:(hexo) ✗ mv .git ../
➜  github.io ✗ hexo init 
➜  github.io ✗ mv ../.git ./
```

### 目录结构

目录结构如下：
- `_config.yml` 是 Hexo 的配置文件，你的自定义配置都应该在该文件中修改
- `scaffolds/` 文件夹保存了 hexo 生成文章的模版文件
- `source/` 文件夹主要保存文章的 Markdown 源文件
- `themes/` 文件夹保存主题的源码
- `public/` 生成的静态文件，包括图片、Tags、Categories 等都保存在这里。执行 `hexo generate` 生成，`hexo clean` 则会清空该文件夹。部署到 GitHub Pages 上的亦即该文件夹下的内容。

```sh
➜  github.io git:(hexo) ls -F         
_admin-config.yml  node_modules/      public/            themes/
_config.yml        package-lock.json  scaffolds/         yarn.lock
db.json            package.json       source/
```

### 部署到 GitHub

最开始 source/ 中会自带一篇介绍 Hexo 的文章，我们可以直接拿过来测试部署是否成功。

Hexo 支持本地启动，命令为：
```sh
hexo s
```
启动后访问 `http://localhost:4000`，你应该可以看到自带的那篇文章。

为部署到 GitHub Pages 上，我们还需要做两件事：**安装依赖**和**修改配置**。

在项目所在目录下执行：

```sh
npm install hexo-deployer-git --save
```

修改 `_config.yml` 将 repo 指定为自己的仓库地址，发布到的分支默认为 master：

```yml
deploy:
  type: git
  repo: git@github.com:username/username.github.io.git
  branch: master
```

现在，执行如下命令即可将发布文章：
```sh
// hexo generate -deploy
hexo g -d
```
访问你的网址 `https://${GitHub username}.github.io` 去见证成果吧！

## 更换主题

Hexo 自带的默认主题是 landscape，不过我们可以从 [Hexo主题库](https://hexo.io/themes/) 或 GitHub 上找到自己喜欢的主题。Next 是 Hexo 最受欢迎的主题之一，下面我将使用它像大家演示如何更换主题。

目前 Next 主题已经升级到 v6.0 版本以上，GitHub 上的仓库从原先的旧版本（v5.1.4 及以下）：`https://github.com/iissnan/hexo-theme-next` 迁移到了新仓库中：`https://github.com/theme-next/hexo-theme-next`，目前版本是 v6.6.0。如果安装的是 v5 版本，在启动时会提示升级。

**下载源码**，直接将 Next 项目克隆到 Hexo 目录的 `themes/` 文件夹下
```sh
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

打开 Hexo 的 `_config.yml` **启用 Next 主题**
```yml
## Themes: https://hexo.io/themes/
# theme: landscape
theme: next
```

v6.0 版本以上的 [Next官方文档](https://theme-next.org/docs/getting-started/) 也做了迁移，描述的很详细，包含了主题设置、三方服务和插件的配置等。所要找的 `themes/next/_config.yml` 中的配置项在官网上都能找到。

// todo 图片

## 更换域名

未完待续。。。

## ghost-to-hexo-migrater

我自己用 Python 写了一个迁移 Ghost 博客至 Hexo 的程序，项目已经提交到 [GitHub]()。

主要做了如下事情：
- 通过 Ghost API 将 Ghost 博客上的所有文章和图片下载到本地。
- 给每篇文章加上 Hexo 规定的 header，包括 Title、创建日期、Tags；由于 Ghost 博客不支持分类，所以 categories 保留默认为空。
- 按照 Hexo 的规定，创建与文章标题一致的文件夹用于存放文章中的图片，并更换文章中图片的 Markdown 链接。

![7292889b7bb90233a1ac2dd1ac2cd97a.png](4.png)

## 参考

- [Hexo官网](https://hexo.io/zh-cn/index.html)
- [Next官方文档](https://theme-next.org/docs/getting-started/) 
- [GitHub Pages部署个人博客（Hexo篇）—— 掘金](https://juejin.im/post/5acf02086fb9a028b92d8652#heading-17)

