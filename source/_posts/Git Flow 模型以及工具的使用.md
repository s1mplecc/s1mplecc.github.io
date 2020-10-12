---
title: Git Flow 模型以及工具的使用
date: 2018-07-03T08:11:51.000Z
tags: [Git]
categories: [Concepts]
---
## Git Flow 是什么？

> 2010 年 5 月，在一篇名为 “[A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)” 的博文中，Vincent Driessen 介绍了一种构建在 Git 之上的软件开发模型。**通过利用 Git 创建和管理分支的能力，为每个分支设定具有特定的含义名称，并将软件生命周期中的各类活动归并到不同的分支上，实现了软件开发过程不同阶段的相互隔离**。这种软件开发的活动模型被 Vincent 称为 “Git Flow”

## Git Flow 流程图
这是 Vincent 博文中的 Git FLow 流程图：从右向左看，从上到下看。
![git-flow-model](/0.png)

## Git Flow Branches

> Git Flow 的核心就是 **Branch**，通过在项目的不同阶段对 Branch 的不同操作（包括但不限于 create、merge、rebase 等）来实现一个完整的高效率的工作流程。Git Flow Branches 主要分为两大类：**Main Branchs（主分支）** 和 **Supporting branches（辅助分支）**。 其中 Main Branchs 中又包含了 **Master** 和 **Develop**，而 Supporting branches 中包含了 **Feature**、**Release**、**Hotfix** 以及其他自定义分支

### Master

- **master 分支上存放的是最稳定的正式版本**，并且该分支的代码应该是随时可在生产环境中使用的代码（**Production Ready state**）。当一个版本开发完毕后，产生了一份新的稳定的可供发布的代码时，master 分支上的代码要被更新。同时，每一次更新，都需要在 master 上打上对应的版本号（tag）

- **任何人不允许在 master 上进行代码的直接提交，只接受其他分支的合入**。原则上 master 上的代码必须是合并自经过多轮测试且已经发布一段时间且线上稳定的 release 分支（预发分支）

### Develop

- **develop 分支是主开发分支，其上更新的代码始终反映着下一个发布版本需要交付的新功能**。当 develop 分支到达一个稳定的点并准备好发布时，应该从该点拉取一个 release 分支并附上发布版本号。也有人称 develop 分支为 “integration branch”，因为会基于该分支和持续集成工具做自动化的 Nightly builds

- develop 分支接受其他 Supporting branches 分支的合入，最常见的就是 feature 分支，开发一个新功能时拉取新的 feature 分支，开发完成后再并入 develop 分支。需要注意的是，**合入 develop 的分支必须保证功能完整，不影响 develop 分支的正常运行**。

### Feature

- feature 分支又叫做功能分支，一般命名为 feature/xxx，用于**开发即将发布版本或未来版本的新功能或者探索新功能**。该分支通常存在于开发人员的本地代码库而不要求提交到远程代码库上，除非几个人合作在同一个 feature 分支开发。关于这点，ThoughtWorks 洞见上有一篇文章 “[Gitflow 有害论](https://insights.thoughtworks.cn/gitflow-consider-harmful/)” 做了非常有意思的阐述，文章下的评论也异常激烈。也许该文章的名字可能有失偏颇，但文章的本意以及评论传达了一个观点：**feature 分支要求足够细粒度以避免成为 long-lived branch，应当小步小步 merge 而不是一次 merge 大量代码**

- feature 分支只能拉取自 develop 分支，开发完成后要么合并回 develop 分支，要么因为新功能的尝试不如人意而直接丢弃

### Release

- release 分支又叫做预发分支，一般命名为 release/1.2（后面是版本号），**该分支专为测试—发布新的版本而开辟，允许做小量级的 Bug 修复和准备发布版本的元数据信息（版本号、编译时间等）**。通过创建 release 分支，使得 develop 分支得以空闲出来接受下一个版本的新的 feature 分支的合入

- release 分支需要提交到服务器上，交由 QA 进行测试，并由 Dev 修复 Bug。同时根据该分支的特性我们可以部署自动化测试以及生产环境代码的自动化更新和部署

- release 分支只能拉取自 develop 分支，合并回 develop 和 master 分支

### Hotfix

- hotfix 分支又叫热修复分支，一般命名为 hotfix/1.2.1（后面是版本号），**当生产环境的代码（master 上代码）遇到了严重到必须立即修复的缺陷时，就需要从 master 分支上指定的 tag 版本（比如 1.2）拉取 hotfix 分支进行代码的紧急修复，并附上版本号（比如 1.2.1）**。这样做的好处是不会打断正在进行的 develop 分支的开发工作，能够让团队中负责 feature 开发的人与负责 hotfix 的人并行、独立的开展工作

- hotfix 分支只能从 master 上拉取，测试通过后合并回 master 分支和 develop 分支

## Git Flow 工具

> 一旦使用 Git Flow 模型，那么对分支的命令操作必然是频繁且重复的，所幸是有人开发了 Git Flow Script 工具帮助我们简化命令，[GitHub官网](https://github.com/nvie/gitflow)

- 安装

    ```
    // OS X
    brew install git-flow
    
    // Windows
    wget -q -O - --no-check-certificate https://github.com/nvie/gitflow/raw/develop/contrib/gitflow-installer.sh | bash
    ```
    
- 使用`git flow init`进行初始化，会询问你分支的命名，使用默认的即可。初始化完成后自动切换到了 develop 分支

    ```
    ➜  demo git:(master) ✗ git flow init
    No branches exist yet. Base branches must be created now.
    Branch name for production releases: [master] 
    Branch name for "next release" development: [develop] 

    How to name your supporting branch prefixes?
    Feature branches? [feature/] 
    Release branches? [release/] 
    Hotfix branches? [hotfix/] 
    Support branches? [support/] 
    Version tag prefix? []

    ➜  demo git:(develop) ✗ 
    ```

- 下面使用 feature 分支演示 Git Flow 功能。`git flow feature start my-feature`该命令用于新建一个 feature 分支。可以看到，基于 develop 的 feature/my-feature 分支已被创建，并且自动切换到该分支

    ```
    ➜  demo git:(develop) ✗ git flow feature start my-feature
    Switched to a new branch 'feature/my-feature'

    Summary of actions:
    - A new branch 'feature/my-feature' was created, based on 'develop'
    - You are now on branch 'feature/my-feature'

    Now, start committing on your feature. When done, use:

         git flow feature finish my-feature

    ➜  demo git:(feature/my-feature) ✗ 
    ```
    在该功能分支下我创建了 a.txt 文件并 commit 到本地仓库，下面演示结束 feature 分支
    
- `git flow feature finish my-feature`该命令用于将 feature/my-feature 分支合并入 develop 分支并删除该分支。此时已切换到了 develop 分支

    ```
    ➜  demo git:(feature/my-feature) git flow feature finish my-feature
    Switched to branch 'develop'
    Updating 6c6e4cb..345a380
    Fast-forward
     a.txt | 0
     1 file changed, 0 insertions(+), 0 deletions(-)
     create mode 100644 a.txt
    Deleted branch feature/my-feature (was 345a380).

    Summary of actions:
    - The feature branch 'feature/my-feature' was merged into 'develop'
    - Feature branch 'feature/my-feature' has been removed
    - You are now on branch 'develop'

    ➜  demo git:(develop) 
    ```

- release 和 hotfix 的命令使用和 feature 一样。需要注意的是它们实际的操作细节会有一些区别，比如 release finish 时会打上版本号，以及合并入 master 和 develop 两个分支

    ```
    ➜  demo git:(release/1.2) git flow release finish 1.2 
    ...
    
    Summary of actions:
    - Latest objects have been fetched from 'origin'
    - Release branch has been merged into 'master'
    - The release was tagged '1.2'
    - Release branch has been back-merged into 'develop'
    - Release branch 'release/1.2' has been deleted

    ➜  demo git:(master)
    ```
    
- 还有将分支 push 到远程代码仓库和拉取远程代码仓库分支的命令

    ```
    ➜  demo git:(release/1.2) git flow release publish 1.2 
    ➜  demo git:(release/1.2) git flow release pull 1.2 
    ```
    
## Conclusion

> Git Flow 模型通过不同的分支从源代码管理的角度对软件开发活动进行了约束，为我们的软件开发提供了一个可供参考的管理模型。Git Flow 模型让代码仓库保持整洁，让小组各个成员之间的开发相互隔离，能够有效避免处于开发状态中的代码相互影响而导致的效率低下和混乱。但同时，不同的开发团队存在不同的文化，在不同的项目背景情况下都可能**根据该模型进行适当的精简或扩充**。

## References

- [A successful Git branching model —— Vincent Driessen](https://nvie.com/posts/a-successful-git-branching-model/)
- [Git Flow流程 —— 掘金](https://juejin.im/entry/5b7fe2745188253010325bf6)
- [敏捷实践系列(四)：代码管理流程](http://deshui.wang/%E6%95%8F%E6%8D%B7/2015/10/27/sourcecode-management)
- [Git Flow的使用 —— 简书](https://www.jianshu.com/p/36292d36e41d)
- [Git Flow有害论 —— ThoughtWorks洞见](https://insights.thoughtworks.cn/gitflow-consider-harmful/)