---
title: Git 命令总结
date: 2017-11-28T03:03:52.000Z
tags: ['Git']
categories: [Concepts]
---
## Preface

> Git是目前世界上最先进的分布式版本控制系统，Linus花了两周时间用C写出来的。。。相比较CVS及SVN这种老式的集中式版本控制系统，要更加灵活方便，也更安全稳定。强烈推荐[Git教程-廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

- **集中式**：版本库是放在中央服务器中，需要联网才能工作
- **分布式**：没有中央服务器，每个人电脑上有个完整的版本仓库，可以本地修改代码再联网提交到Git服务器上，Git服务器作用仅仅“交换”大家的修改即代码同步。Git还拥有极其强大的分支管理。

## Git Commands

- 常用命令pdf文档 [git-cheatsheet](https://dir.s1mple.xyz/git-cheatsheet.pdf)

### Create Repository

> 始终要记住你的本地和Git服务器上都存在版本仓库，你的日常工作就是做这两个仓库的同步而已。有两种方式创建版本库

- 直接从Git服务器上克隆一个仓库下来，可以通过**ssh**或**http**的方式。ssh需要将你的公钥存放在Git服务器上，而http不需要但是速度会比ssh慢。
    
    ```
    git clone git@github.com:s1mpleinfo/MyHashMap.git
    git clone https://github.com/s1mplecc/MyHashMap.git
    ```
    
  从远程库克隆下来的是完整的版本库，含有`.git`文件夹，其下的`config`文件夹包括了本仓库的配置信息。包含了url、远程库的名字以及分支等
    ```
    [remote "origin"]
        url = git@github.com:s1mpleinfo/MyHashMap.git
        fetch = +refs/heads/*:refs/remotes/origin/*
    [branch "master"]
        remote = origin
        merge = refs/heads/master
    ```

- `git init`将当前目录初始化为Git可以管理的仓库。可以看到在当前目录下生成了`.git`文件夹。

  初始化后的仓库需要绑定Git服务器上的版本库，**origin**是版本库默认的名字
    ```
    git remote add origin git@github.com:s1mpleinfo/MyHashMap.git
    ```

    
### Submit Changes

- `git status`查看当前版本库的状态

- `git add .`将修改提交到暂存区。`.`表示当前目录下所有修改了的文件，当然也可以指定某个文件。

- 将暂存区的所有内容提交到当前分支，默认分支即Git为我们自动创建的**master**分支。`-m`参数后面接提交的message
    
    ```
    git commit -m "balabala"
    ```

- 将当前分支提交到远程库中，`-u`第一次使用时将本地的master分支绑定到远程库的master分支，以后可以省略不加
    
    ```
    git push -u origin master
    ```
    
- 从远程库拉取修改

    ```
    git pull origin master
    ```

    #### Tips

- 说到`add`不得不提`.gitignore`这个文件，它的作用是忽略一些文件的提交，比如IDEA工具生成的`.idea`文件、编译生成的`target`文件等等。关于`.gitignore`的编写github上都有现成的模版。另外IDEA上可以安装**ignore**插件。

- 实际上日常中我们用到最多的应该就是上述命令了。本质上本地与远程版本库的同步就是记录下每次的**修改信息**。Git会帮我们记录下每次提交的修改，方便我们进行同步、回退以及合并。

### Commit History

> 正是Git帮我们记录了每次提交的修改，我们可以通过命令查看历史信息，甚至每次做了什么修改

- `git log`查看提交历史，每条记录有一个唯一的id用于标识，使用时可以使用可以唯一表示的前几位。添加参数`-p <file>/<commit id>`查看指定文件或某次commit的历史信息

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

- `git diff b90b6da`查看某次提交后与当前版本的区别，可以看到我在文本中添加了一行second

    ```
    @@ -1 +1,2 @@
     first
    +second
    ```
    
- `git reflog`查看命令操作历史

### Undo Changes

> 你可能会遇到这种情况，提交了某次修改又后悔了，Git提供了回退到某次提交的操作

- `reset`命令回退到指定的某个版本，`HEAD`表示当前版本，`HEAD^`表示上一个版本。也可以用commit id
     
     ```
     git reset --hard HEAD^
     ```
     
- 如果你修改了某个文件，还没有`add`，可以舍弃这次修改
   
    ```
    git checkout -- <file>
    ```
     
- 如果你已经`commit`了，也可以还原回去

    ```
    git revert <commit id>
    ```

     #### Tips
     
- Git将提交记录（节点）串成一条线，**HEAD**就是一指向当前版本的指针，做回退操作其实就是这个指针在这条线上的节点间移动。牢记：**Git跟踪并管理的是修改，而非文件**
     
- revert与reset的区别是，revert会生成一次新的提交去移除某些提交，在节点线上还是向前走的。而reset是把HEAD指针后移到指定节点，直接从这个节点走出新的一条线。[代码回滚：git reset、git checkout和git revert区别和联系](http://www.cnblogs.com/houpeiyong/p/5890748.html)

### Git Branches

> Git 分支可以参考我的另一篇博客 [Git Flow 模型以及工具的使用](https://s1mple.xyz/git-flow/)。Git Flow 是一种构建在 Git 之上的软件开发模型。通过利用 Git 创建和管理分支的能力，为每个分支设定具有特定的含义名称，并将软件生命周期中的各类活动归并到不同的分支上，实现了软件开发过程不同阶段的相互隔离

## Postscripts

> Git命令还有很多，常用的大概就是这些，如果能将这些使用的很溜，那工作上遇到的问题一般都可以解决了。其实常用命令都可以在IDEA中设置快捷键，使用起来更加方便。


