---
title: Mac 的终极 Shell -- ZSH
date: 2018-07-30T07:46:19.000Z
tags: ['Linux']
categories: [Ops]
---
## 前言

目前常用的 Linux 系统和 OS X 系统的默认 Shell 都是 bash，但是真正强大的 Shell 是深藏不露的 zsh， 这货绝对是马车中的跑车，跑车中的飞行车，史称『终极 Shell』，但是由于配置过于复杂，所以初期无人问津，很多人跑过来看看 zsh 的配置指南，什么都不说转身就走了。直到有一天，国外某位大牛开源出一个能够让你快速上手的 zsh 项目，叫做『oh my zsh』，使得 zsh 逐渐流行起来。

## ZSH

- 安装 zsh

```
brew install zsh
```
安装完后在 Terminal 的设置中设置默认使用 zsh![8F9622E7-901E-46F0-94D7-B5E7AEE04C08](/0.png)

## Oh My Zsh

- oh-my-zsh 是最为流行的 zsh 配置文件，提供了完善的插件体系，相关的文件在`~/.oh-my-zsh/plugins`目录下。[GitHub 链接](https://github.com/robbyrussell/oh-my-zsh)

- 安装 [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
安装完成后`source ~/.zshrc`使之生效

### Configuration

- 在`.zshrc`中添加配置，其中可以自定义`alias`用于简化命令，这里贴出我的配置。`alias -s`意为 define suffix aliases，譬如命令行输入`xxx.gz`就相当于`tar -xzvf xxx.gz`

```
# compress relatived
alias -s gz='tar -xzvf'
alias -s tgz='tar -xzvf'
alias -s zip='unzip'
alias -s bz2='tar -xjvf'

# sublime text
alias subl="open -a 'Sublime Text'"
alias -s php=subl
alias -s rb=subl
alias -s html=crm
alias -s xml=subl
alias -s txt=subl
alias -s hbs=subl

# macdown
alias macdown="open -a MacDown"
alias -s md=macdown

# git relatived
alias gpom='git push -u origin master'
alias grao='git remote add origin'

# others
alias his='history'
alias la='ls -a'
alias ll='ls -alh'
```

- 添加如下配置后，可以使用`subl a.txt`或者直接输入`a.txt`通过 Sublime 打开文本文件

```
alias subl="open -a 'Sublime Text'"
alias -s txt=subl
```
- 另外，添加了两个常用 Git 命令的缩写，其实 Oh My Zsh 已经包含了许多 Git 的简写命令，比如常用的`gst`即`git status`的简写。可以使用命令`alias`查看还有哪些简写命令

### Plugins

- oh-my-zsh 有很多提升命令行效率的实用工具，我目前使用了如下四个插件，确实在使用过程中体验到效率的提升。安装完成后在`.zshrc`中打开插件，其中 git 是默认安装并开启的插件，前面也提到有许多简写命令，这里不多赘述，请自行查阅

```
plugins=(
  git
  Z
  zsh-syntax-highlighting
  zsh-autosuggestions
)
```

#### Z

- 功能和 autojump 类似，使用`z {path}`实现目录间快速跳转，并且作为 oh-my-zsh 的内置组件，只需要在`.zshrc`中打开即可。[GitHub 链接](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/z)

- 如图，Z 插件会记录你前往的目录，并根据进入次数计算权重（`z -r`查看权重），以后再进入该目录只用`z ddd`，会进行**模糊匹配**，前往**权重最高**的目录
![678A0E3A-EDE9-4D9B-AA39-A6E14DC6E2EC](/1.png)

#### zsh-syntax-highlighting

- 终端输入的命令正确会**绿色高亮**显示，输入错误会显示红色，用于在执行命令前检查是否有语法错误。[GitHub 链接](https://github.com/zsh-users/zsh-syntax-highlighting)
![82881CCA-A343-45D1-9A1D-4DD8E77E1B78](/2.png)

- 安装，完成后在`.zshrc`中添加

```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

#### zsh-autosuggestions

- 效率神器。如图输入命令时，会给出建议的命令（灰色部分）按键盘 → 补全。[GitHub 链接](https://github.com/zsh-users/zsh-autosuggestions)
![8B1B4FDE-F810-4E90-B20A-37274ABDDFDE](/3.png)

- 安装，完成后在`.zshrc`中添加

```
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

## 参考

- [终极 Shell——ZSH - 知乎专栏](https://zhuanlan.zhihu.com/p/19556676)
- [一些命令行效率工具](http://wulfric.me/2015/08/zsh/)