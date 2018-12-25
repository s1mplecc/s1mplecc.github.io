---
title: Git 进阶教程
date: 2018-09-18T02:02:04.000Z
tags: ['Git']
categories: []
---
## 基本概念

> 你的本地仓库由 Git 维护的三棵「树」组成。第一个是你的**工作目录（Working dir）**，它持有实际文件，即你所见的；第二个是**缓存区（Stage or Index）**，它像个缓存区域，临时保存你的改动；第三个是**提交历史（Commit history）**，包含的 HEAD 指针指向你最近一次 commit 的引用。关于工作区和缓存区概念可参考 [廖雪峰的教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013745374151782eb658c5a5ca454eaa451661275886c6000)

![687474703a2f2f6d61726b6c6f6461746f2e6769746875622e696f2f76697375616c2d6769742d67756964652f62617369632d75736167652e737667-1](/0.svg)

上面的四条命令在工作目录、stage 缓存（也叫做索引）和 commit 历史之间复制文件。

- `git add files` 把工作目录中的文件加入 stage 缓存
- `git commit` 把 stage 缓存生成一次 commit，并加入 commit 历史
- `git reset -- files` 撤销最后一次 git add files，你也可以用 git reset 撤销所有 stage 缓存文件
- `git checkout -- files` 把文件从 stage 缓存复制到工作目录，用来丢弃本地修改

## Merge

`git merge` 命令把不同分支合并起来。合并前，索引必须和当前提交相同。**`git merge other` 用于将 other 分支上的提交合并到当前分支**。

如果另一个分支是当前提交的祖父节点，那么合并命令将什么也不做。另一种情况是如果当前提交是另一个分支的祖父节点，就导致 **fast-forward** 合并，HEAD 指针只是简单的移动。
![687474703a2f2f6d61726b6c6f6461746f2e6769746875622e696f2f76697375616c2d6769742d67756964652f6d657267652d66662e737667](/1.svg)

否则就是一次真正的合并。默认把当前提交（_ed489_ 如下所示）和另一个提交（_33104_）以及他们的共同祖父节点（_b325c_）进行一次[三方合并](https://en.wikipedia.org/wiki/Merge_(version_control)#Three-way_merge)。结果是先保存当前目录和索引，然后和父节点 _33104_ 一起做一次**新提交**。
![687474703a2f2f6d61726b6c6f6461746f2e6769746875622e696f2f76697375616c2d6769742d67756964652f6d657267652e737667](/2.svg)

## Cherry Pick

`git cherry-pick` 命令「复制」一个提交节点并在当前分支做一次完全一样的新提交。

![687474703a2f2f6d61726b6c6f6461746f2e6769746875622e696f2f76697375616c2d6769742d67756964652f6368657272792d7069636b2e737667](/3.svg)

## Rebase

`git rebase` 是合并命令的另一种选择。merge 把两个分支合并进行一次新提交，提交历史不是线性的。rebase 在当前分支上重演另一个分支的历史，**提交历史是线性的**。

![687474703a2f2f6d61726b6c6f6461746f2e6769746875622e696f2f76697375616c2d6769742d67756964652f7265626173652e737667](/4.svg)

上面的命令都在 _topic_ 分支中进行，执行 `git rebase master` 意味着将 _topic_ 分支基于  _master_ 分支进行重演，在 _da985_ 后附加对应于 _169a6_ 和 _2c33a_ 的两次新提交，所以提交历史是线性的。

### Difference between rebase & merge

- `git rebase master`
    ```sh
          A---B topic
         /
    D---E---F---G master


                  A'--B' topic
                 /
    D---E---F---G master
    ```
    
- `git merge master`
    ```sh
          A---B topic
         /
    D---E---F---G master


          A---B---H topic
         /       / 
    D---E---F---G master
    ```

> rebase 最大的好处是你的项目历史会非常整洁的线性，它不像 merge 那样引入不必要的合并提交。但同时，rebase 也会带来两个后果：安全性和可跟踪性。使用时要遵守 **rebase 的黄金法则：绝不要在公共的分支上使用它**。可跟踪性是指 rebase 不会有合并提交中附带的信息，你看不到 _topic_ 分支中并入了 _master_ 的哪些更改。

## Reset

`git reset` 命令把当前分支指向另一个位置，并且有选择的变动工作目录和索引。如果**你的更改还没有共享给别人**，reset 是撤销这些更改的简单方法。当你开发一个功能的时候发现「糟糕，我做了什么？我应该重新来过！」时，reset 就像是 `go-to` 命令一样。

![687474703a2f2f6d61726b6c6f6461746f2e6769746875622e696f2f76697375616c2d6769742d67756964652f72657365742d636f6d6d69742e737667](/5.svg)

还可以通过传入参数来修改你的缓存区或工作目录：

- `--soft` 缓存区和工作目录都不会被改变
- `--mixed` 默认选项。缓存区同步到你指定的提交，但工作目录不受影响
- `--hard` 缓存区和工作目录都同步到你指定的提交

把这些参数想成定义 `git reset` 操作的**作用域**就容易理解多了。拿上图举例来说，如果执行 `git reset --soft HEAD~3`，那么文件不会被修改，后三次提交被 `git add` 添加到暂存区，相当于只撤销了 `git commit`；`git reset --mixed HEAD~3` 则文件不会被修改，但后三次提交没有添加到暂存区；`git reset --hard HEAD~3` 则文件会被回退到 *b325c* 时的状态。

## Revert

**revert 撤销一个提交的同时会创建一个新的提交**。这是一个安全的方法，因为它不会重写提交历史。比如，下图执行`git revert HEAD~2`会创建一个新的提交来撤销前两个提交的更改，然后把新的提交加入提交历史。

![06](/6.svg)

相比 `git reset`，它不会改变现在的提交历史。因此，**`git revert` 可以用在公共分支上，`git reset` 应该用在私有分支上**。你也可以把 `git revert` 用作撤销已经 push 的更改，而 `git reset` 用来撤销已经 commit 但没有 push 的更改，或者 `git reset HEAD` 撤销没有 commit 的更改。

## Checkout

**checkout 命令用于从历史提交（或者暂存区域）中拷贝文件到工作目录，也可用于切换分支**。

当给出一个（本地）分支时，那么 _HEAD_ 指针将指向那个分支，也就是说，我们「切换」到那个分支了。**stage 缓存和工作目录中的内容会和 _HEAD_ 对应的提交节点一致**。

![a](/7.svg)

当指定的是一个标签、远程分支、SHA-1 值或者是像 master~3 类似的东西，就得到一个**匿名分支**，又称作 **detached HEAD（被分离的 HEAD 标识）**。这样可以很方便地在历史版本之间互相切换。比如说你想要编译 1.2.1 版本，你可以运行 `git checkout v1.2.1`（这是一个标签 tag
，而非分支名），编译，安装，然后切换回原先分支。

![b](/8.svg)

#### Detached HEAD

当提交操作涉及到 detached HEAD 时，其行为会略有不同。_HEAD_ 处于分离状态（不依附于任一分支）时，提交操作可以正常进行，但是不会更新任何已命名的分支。你可以认为这是在更新一个匿名分支。

![c](/9.svg)

一旦此后你切换到别的分支，比如说 _master_，那么这个提交节点（可能）再也不会被引用到，将会被丢弃掉。

![d](/10.svg)

但是，如果你想保存这个状态，可以用命令 `git checkout -b name` 来创建一个新的分支。

![e](/11.svg)

## References

- [git-recipes —— GitHub 8k star 的高质量教程](https://github.com/geeeeeeeeek/git-recipes)
- [Merging vs. Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)
- [Resetting, Checking Out & Reverting](https://www.atlassian.com/git/tutorials/resetting-checking-out-and-reverting)