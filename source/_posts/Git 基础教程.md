---
title: Git 基础教程
date: 2017-11-28T03:03:52.000Z
tags: [Git]
categories: [Concepts]
---
## 前言

Git 是目前世界上最先进的分布式版本控制系统。相比较CVS及SVN这种老式的集中式版本控制系统，要更加灵活方便，也更安全稳定。**集中式**是指版本库是放在中央服务器中，需要联网才能工作。**分布式**是指没有中央服务器，每个人电脑上有个完整的版本仓库，可以本地修改代码再联网提交到Git服务器上，Git服务器作用仅仅“交换”大家的修改即代码同步。此外，Git还拥有极其强大的分支管理功能。

## Git 命令

本文主要介绍基础的 Git 命令，更加复杂的 Git 命令可以查看：[Git 进阶教程](https://s2mple.xyz/2018/09/18/Git%20%E8%BF%9B%E9%98%B6%E6%95%99%E7%A8%8B/)。另外 Git 的常用命令文档可以参考：[git-cheatsheet](https://education.github.com/git-cheat-sheet-education.pdf)。

### 创建仓库

始终要记住你的**本地和Git服务器上都存在版本仓库（repository）**，Git实现代码同步实际上是同步你的本地仓库和远程仓库。有两种方式创建仓库：1. 从服务器上克隆一个仓库；2. 初始化一个本地仓库。

从Git服务器上**克隆（clone）**一个仓库下来，可以通过**ssh**或**http**的方式。ssh需要将你的公钥存放在Git服务器上，而http不需要但是速度相较ssh要慢。
    
```
git clone git@github.com:s1mplecc/MyHashMap.git
git clone https://github.com/s1mplecc/MyHashMap.git
```

从远程库克隆下来的是完整的版本库，含有`.git`文件夹，其下的`config`文件夹包括了本仓库的配置信息。包含了url、远程库的名字以及分支等：

```
➜ cat .git/config

[remote "origin"]
    url = git@github.com:s1mplecc/MyHashMap.git
    fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
    remote = origin
    merge = refs/heads/master
```

第二种方式，`git init`是将当前目录初始化为Git可以管理的仓库。会在当前目录下生成了`.git`文件夹。

如果要将本地仓库推到服务器上，需要现在服务器上创建一个repository，然后在本地目录下绑定服务器上的repository，origin是版本库默认的名字：

```
git remote add origin git@github.com:s1mplecc/MyHashMap.git
```
    
### 提交变更

当你对版本库中的文件做了变更（增删改），可以使用`git status`命令查看当前版本库的状态：

```
➜  github.io git:(hexo) ✗ git status
On branch hexo
Your branch is ahead of 'origin/hexo' by 9 commits.
  (use "git push" to publish your local commits)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   "source/_posts/ES6\346\226\260\347\211\271\346\200\247.md"
    modified:   "source/_posts/Git Flow \346\250\241\345\236\213\344\273\245\345\217\212\345\267\245\345\205\267\347\232\204\344\275\277\347\224\250.md"

no changes added to commit (use "git add" and/or "git commit -a")
```

`git add .`或者`git add all`将所有修改提交到暂存区。`.`表示当前目录下所有修改了的文件，当然也可以指定某个文件。

将暂存区的所有内容提交到当前分支，默认分支即Git为我们自动创建的**master**分支。`-m`参数后面接提交的message：
    
```
git commit -m "balabala"
```

将当前分支提交到远程库中，`-u`第一次使用时将本地的master分支绑定到远程库的master分支，以后可以省略不加：

```
git push -u origin master
```

从远程库拉取修改：

```
git pull origin master
```

在开发过程中会有频繁的提交，所以日常中我们用到最多的就是上述命令了。本质上本地与远程版本库的同步就是记录下每次的**修改信息**。Git会帮我们记录下每次提交的修改，方便我们进行同步、回退以及合并。

#### .gitignore

说到`add`不得不提`.gitignore`这个文件，它的作用是忽略一些文件的提交，比如IntelliJ IDEA生成的`.idea`文件、编译生成的`target`文件等等。这类文件不需要提交到服务器污染大家的环境。关于`.gitignore`的编写github上都有现成的模版。另外IDE也可以安装相应的gitignore插件。

### 查看提交历史

正是Git帮我们记录了每次提交的修改，我们可以通过命令查看历史信息，以及每次做了什么修改。

`git log`查看提交历史，每条记录有一个唯一的id用于标识，使用时可以使用可以唯一表示的前几位。添加参数`-p <file>/<commit id>`查看指定文件或某次commit的历史信息：

```
commit 24345a1a74184f99eb32202bfcc1cd0fd6ac945d (HEAD -> master)
Author: s1mple <s1mple@macpro.local>
Date:   Wed Nov 29 11:33:56 2017 +0800

    second commit

commit b90b6da1573d55ef59a900d435d903e41667f61e
Author: s1mple <s1mple@macpro.local>
Date:   Wed Nov 29 11:31:43 2017 +0800

    first commit
```

`git diff b90b6da`查看某次提交后与当前版本的区别，可以看到我在文本中添加了一行second：

```
@@ -1 +1,2 @@
 first
+second
```
    
`git reflog`查看命令操作历史。

### 撤销变更

如果你在提交了某次变更后又想撤销这次变更，Git也提供了撤销和回退的功能。Git将提交记录（节点）串成一条线，**HEAD**就是一指向当前版本的指针，做回退操作其实就是这个指针在这条线上的节点间移动。牢记：**Git跟踪并管理的是修改，而非文件**。

`reset`命令回退到指定的某个版本，`HEAD`表示当前版本，`HEAD^`表示上一个版本。也可以用具体的 commit id：
     
```
git reset --hard HEAD^
```

如果你修改了某个文件，但还没有`add`，可以直接舍弃这次修改：

```
git checkout -- <file>
```

如果你已经`commit`了，也可以还原回去：

```
git revert <commit id>
```
     
revert与reset的区别在于，revert会生成一次新的提交去移除某些提交（相当于反向操作），在节点线上还是向前走的。而reset是把HEAD指针后移到指定节点，直接从这个节点走出新的一条线。可以参考[代码回滚：git reset、git checkout和git revert区别和联系](http://www.cnblogs.com/houpeiyong/p/5890748.html)。

## 参考

- [git-cheatsheet](https://education.github.com/git-cheat-sheet-education.pdf)
- [Git教程 —— 廖雪峰](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013745374151782eb658c5a5ca454eaa451661275886c6000)
