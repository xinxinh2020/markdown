[TOC]

# 深入浅出一篇就够之:Git

声明：本文不是0基础使用教程，不介绍如何安装git以及最基本的git操作，本文侧重于介绍git的核心原理和思想，让读者学会“thinking in git”。本文尝试用最浅显的语言来描述git最本质的原理，如果读者有过git的使用经验阅读本文可能会有更深的体会，但即使没有，看完本文对git的核心思想应该也会有所理解。



## 当我checkout时，我在做什么？

checkout这个词，中文一般翻译为检出，但这个词在中文语境下很难单从字面上知道它具体是什么意思，因为我们几乎没有使用这个词的习惯，不信你仔细想想，除了在git等版本控制工具中我们在其它地方是不是很少见到这个词？我们知道checkout有一个很重要的作用就是用来切换分支，那么为什么git不使用switch这种更好理解的词而要用checkout来描述这个操作呢？其实checkout这个命令暗藏的玄机，远远不止切换分支这么简单。

我们知道git有个本地仓库的概念(其实就是.git目录，下文会细讲)，checkout本质上是从这个仓库中拉出相应分支的数据(代码)的过程。checkout的时候，并不是简单的把本地仓库(再次强调，本地仓库实际上就是.git目录)中的文件拷贝出来，因为git并不是把我们各个分支的文件以原始的方式存放到这个本地仓库中的，而是对数据进行了压缩编码等处理后放进去的，所以当我们切换分支时，git需要把这个分支的数据从本地仓库里解压解码后才能放到我们的工作目录，这个过程Git就称之为checkout。现在我们就很好理解，为什么不用switch呢？



因为当我们说切换的时候，可能会带来一种误解，一般是把当前的任务保存起来，再把新的任务(分支)加载进来，但是checkout实际上并没有保存当前任务状态的功能(保存状态实际上commit做的事)，可以简单的理解为它直接把新分支从本地仓库中加载进来直接覆盖掉当前分支！这就是为什么我们把修改add到暂存区以后，如果不commit一下就切换分支，并且修改和要切换的分支发生冲突时，就会失败，，所以少年，请先保存一下(commit)呀：

```shell
PS D:\workspace\vscode\starlight-test> git checkout job-management
error: Your local changes to the following files would be overwritten by checkout:
        env/uat/js.properties
        impl/login.js
        specs/login.cpt
        specs/login.spec
Please commit your changes or stash them before you switch branches.
Aborting
```





