---
title: npm、nrm、nvm的使用以及Mac下如何管理Node版本
date: 2017-12-27T03:01:29.000Z
tags: ['JavaScript', 'Node.js']
categories: [Ops]
---
## npm

> npm(node依赖包管理器，node package manager)，官方或权威机构提供了一个公共的软件源（类似 Homebrew、yum），可方便我们下载并管理 JavaScript 依赖

安装了 node 就包含了 npm 的安装

安装依赖，`-g`参数表示全局安装，不带参数则安装在当前目录

```
npm install lodash
npm install -g vue-cli
```

可查看已安装的全局依赖（默认安装到 `/usr/local/lib/node_modules` 目录下），`--depth`显示的树深度，不加`-g`参数查看当前目录安装的依赖

```
➜ npm ls -g --depth=0
/usr/local/lib
├── npm@5.6.0
├── nrm@1.0.2
└── vue-cli@2.9.2
```

卸载依赖

```
npm uninstall lodash
```

搜索某个依赖，以及显示某个依赖的具体信息，经测试，只有官方的源可以`search`，其他源未做相应支持

```
npm search lodash
npm [view|show|info] lodash
```

`npm config ls`查看配置信息，`-l`参数查看详细配置

```
➜ npm config ls
; cli configs
metrics-registry = "https://registry.npmjs.org/"
scope = ""
user-agent = "npm/5.6.0 node/v9.3.0 darwin x64"

; userconfig /Users/s1mple/.npmrc
registry = "https://registry.npmjs.org/"

; builtin config undefined
prefix = "/usr/local"

; node bin location = /usr/local/Cellar/node/9.3.0_1/bin/node
; cwd = /Users/s1mple/WebstormProjects/vue/table
; HOME = /Users/s1mple
; "npm config ls -l" to show all defaults.
```

默认软件源为国外的官方软件源，速度较慢，可修改为国内软件源，比如淘宝

```
npm config set registry https://registry.npm.taobao.org
```

npm 还可以登录，需要先在官网注册一个账号，然后 `npm login` 登录，就可以将自己写的模块 `publish` 上去，有点类似于 Docker 的个人仓库，网站地址为 `https://www.npmjs.com/~{username}`。另外，还有一些命令简写，比如`i`代替 `install`，`rm` 代替 `uninstall`，执行 `npm -l` 查看那些命令下面有哪些 alias


## nrm

> nrm(npm注册处管理器，npm registry manager)，方便我们切换软件源

全局安装 nrm

```
npm i -g nrm
```

nrm 内置了一些软件源，带`*`星号的表示当前使用的软件源，也可以使用 `nrm current` 查看

```
➜ nrm ls

* npm ---https://registry.npmjs.org/
  cnpm --http://r.cnpmjs.org/
  taobao https://registry.npm.taobao.org/
  nj ----https://registry.nodejitsu.com/
  rednpm http://registry.mirror.cqupt.edu.cn/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm http://registry.enpmjs.org/
```

切换软件源，比如换成淘宝的，相当于手动修改了`~/.npmrc`中的`registry`配置

```
➜ nrm use taobao

    Registry has been set to: https://registry.npm.taobao.org/
```


可以执行`nrm test`测试网络延迟

```
➜  ~ nrm test npm

* npm ---2608ms

➜  ~ nrm test taobao

  taobao 246ms

➜  ~ nrm test cnpm

  cnpm --256ms
```

`npm home taobao`打开对应源的官方网站

## nvm

> nvm(node版本管理器，node version manager)，可以安装、管理不同版本 node 并进行切换

安装，完成后重启终端

```
curl -ohttps://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash
```
这个脚本克隆了 nvm repository 到`~/.nvm`并在`~/.bash_profile, ~/.zshrc, ~/.profile, ~/.bashrc`中加入环境变量

安装最新的 node

```
➜ nvm install node

...
Now using node v9.3.0 (npm v5.5.1)
Creating default alias: default -> node (-> v9.3.0)
```

或安装指定版本，执行`nvm ls-remote`可列出所有可以安装的版本

```
➜ nvm install 8.9.3
```

查看已安装 node 版本

```
➜ nvm ls
->       v8.9.3
         v9.3.0
         system
default -> node (-> v9.3.0)
node -> stable (-> v9.3.0) (default)
stable -> 9.3 (-> v9.3.0) (default)
```

箭头指向当前使用的版本，切换版本

```
➜ nvm use 9.3.0
Now using node v9.3.0 (npm v5.5.1)
```

还可以指定默认的node版本

```
nvm alias default 8.9.3
```

## Mac管理Node版本

最近有尝试将 Hexo 博客包括 Next 主题进行升级，也是遇到了很多坑，版本进行一点点的改动，都可能导致整个博客网站 run 不起来，无奈之下只能封存了现有的版本，劝自己不要手贱去更新。如果以后有更新的强迫意愿，只能考虑在全新环境下进行博客内容的迁移。

其中我遇到的一个比较头疼的问题就是 node 的版本问题。Hexo 在 node 版本为8或者14时均不能正常运行，`hexo s` 以及发布命令 `hexo g -d` 都跑不起来，只有在 node 版本为10时才可以正常运行。

我之前使用过 `brew install node` 安装了最新的版本14，虽然可以用nvm去管理 node 版本，但由于要想使用 nvm，必须先执行 `[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"` 和 `[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"` 这两条命令（配在了我的 `~/.zshrc` 中），极大影响了终端的打开速度，每打开一个终端都要跑一遍 `.zshrc`，效率难以忍受。再者平时切换 node 版本的频率要远低于执行 Hexo 命令的频率。所以我决定还是回到用Mac的 `brew` 工具去管理 node 版本。


查看可用版本，并安装需要版本
```
➜ brew search node
➜ brew install node@10
```

现在是有两个版本的，node为原先安装的14版本

```
➜ brew list | grep node
node
node@10
```

解绑原先的 node@14，绑定到 node@10，由于 node 自带 npm，需要先删除原有的 npm 依赖，否则会有冲突

```
➜ brew unlink node
➜ rm -rf /usr/local/lib/node_modules/npm
➜ brew link node@10 --force
```

查看是否绑定成功

```
➜ which node
/usr/local/bin/node
➜ node -v
v10.22.1
```

现在 node 版本就切换到10了，新打开的终端中的 node 版本也为10。如果还想切换回 node@14，有两种方法：一是使用 nvm 进行切换，但这样新打开终端 node 版本还是14，适用于短暂的更换 node 版本；第二种办法是使用 `brew unlink` 解绑再绑定 node@14，这种适用于主环境需要14版本的情况。

## 参考

- [npm 官方文档](https://docs.npmjs.com/)
- [nvm Github官方手册](https://github.com/creationix/nvm/blob/master/README.md)
- [OS X中使用brew管理多个node版本](https://mrxf.github.io/2017/04/18/osx-using-the-brew-to-manage-multiple-node-version/)