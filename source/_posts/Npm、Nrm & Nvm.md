---
title: Npm、Nrm & Nvm
date: 2017-12-27T03:01:29.000Z
tags: ['JavaScript', 'Node.js']
categories: []
---
## Npm

> **Npm**(or **Node Package Manager**)，**Node** 依赖包管理器，官方或权威机构提供了一个公共的软件源（类似 **Homebrew、yum**），可方便我们下载并管理 **JavaScript** 依赖

- 安装了 **Node** 就包含了 **Npm** 的安装

- 安装依赖，`-g`参数表示全局安装，不带参数则安装在当前目录

    ```
    npm install lodash
    npm install -g vue-cli
    ```

- 可查看已安装的全局依赖（默认安装到`/usr/local/lib/node_modules`目录下），`--depth`显示的树深度，不加`-g`参数查看当前目录安装的依赖

    ```
    ➜ npm ls -g --depth=0
    /usr/local/lib
    ├── npm@5.6.0
    ├── nrm@1.0.2
    └── vue-cli@2.9.2
    ```
    
- 卸载依赖

    ```
    npm rm lodash
    ```
    
- 搜索某个依赖，以及显示某个依赖的具体信息，另外经测试，只有官方的源可以`search`，其他源未做相应支持

    ```
    npm search lodash
    npm [view|show|info] lodash
    ```

- `npm config ls`查看配置信息，`-l`参数查看详细配置

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

- 默认软件源为国外的官方软件源，速度较慢，可修改为国内软件源，比如淘宝

    ```
    npm config set registry https://registry.npm.taobao.org
    ```

- **Npm** 还可以登录，需要先在官网注册一个账号，然后`npm login`登录，就可以将自己写的依赖`publish`上去，有点类似于 **Docker** 的个人仓库，网站地址为`https://www.npmjs.com/~{username}`

    ```
    https://www.npmjs.com/~s1mple
    ```

- 另外，还有一些命令简写，比如`i`代替`install`，`rm`代替`uninstall`，执行`npm -l`查看那些命令下面有哪些 **alias**

---

## Nrm

> **Nrm**(or **Npm Registry Manager**)，**Npm** 注册处管理器，方便我们切换软件源

- 全局安装 **nrm**

    ```
    npm i -g nrm
    ```

- **nrm** 内置了一些软件源，带`*`星号的表示当前使用的软件源，当然也可以使用`nrm current`查看

    ```
    ➜ nrm ls

    * npm ---- https://registry.npmjs.org/
      cnpm --- http://r.cnpmjs.org/
      taobao - https://registry.npm.taobao.org/
      nj ----- https://registry.nodejitsu.com/
      rednpm - http://registry.mirror.cqupt.edu.cn/
      npmMirror  https://skimdb.npmjs.com/registry/
      edunpm - http://registry.enpmjs.org/
    ```

- 切换软件源，比如换成 **taobao** 的

    ```
    ➜ nrm use taobao
    
        Registry has been set to: https://registry.npm.taobao.org/
    ```
    其实就是修改了`~/.npmrc`中的`registry=...`
    
- 可以执行`nrm test`测试网络延迟

    ```
    ➜  ~ nrm test npm

    * npm ---- 2608ms

    ➜  ~ nrm test taobao

      taobao - 246ms

    ➜  ~ nrm test cnpm

      cnpm --- 256ms
    ```

- `npm home taobao`打开对应源的官方网站

---

## Nvm

> **Nvm**(or **Node Version Manager**)，Github 上一位大神写的**Node** 版本管理器，可以安装、管理不同版本 **Node** 并进行切换

- 安装，完成后重启终端

    ```
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash
    ```
    这个脚本克隆了 **nvm repository** 到`~/.nvm`并在`~/.bash_profile, ~/.zshrc, ~/.profile, ~/.bashrc`中加入环境变量

- 安装最新的 **Node**

    ```
    ➜ nvm install node
    
    ...
    Now using node v9.3.0 (npm v5.5.1)
    Creating default alias: default -> node (-> v9.3.0)
    ```
    
- 或安装指定版本，执行`nvm ls-remote`可列出所有可以安装的版本

    ```
    ➜ nvm install 8.9.3
    ```
    
- 查看已安装 **Node** 版本

    ```
    ➜ nvm ls
    ->       v8.9.3
             v9.3.0
             system
    default -> node (-> v9.3.0)
    node -> stable (-> v9.3.0) (default)
    stable -> 9.3 (-> v9.3.0) (default)
    ```

- 箭头指向当前使用的版本，切换版本

    ```
    ➜ nvm use 9.3.0
    Now using node v9.3.0 (npm v5.5.1)
    ```

- 还可以指定默认的Node版本

    ```
    nvm alias default 8.9.3
    ```

---

## References

- [npm 官方文档](https://docs.npmjs.com/)
- [nvm Github官方手册](https://github.com/creationix/nvm/blob/master/README.md)