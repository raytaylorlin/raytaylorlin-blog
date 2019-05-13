title: Git指南（2）——处理历史与错误
date: 2013-09-17 17:18:08
categories:
- 技术
- 工具
- Git
tags:
- Git
---
上一篇介绍了Git的一些最常用的基本命令，如果只是日常的提交，那些命令已经够用了。但是平时写代码的时候总难免会需要查看你以前写过的代码，或者当你进行了一些误操作的时候，总会希望可以安全地撤销你的操作。本篇将介绍如何处理历史记录，在各个版本之间跳转（签出），以及如何撤销你的之前的提交、更改等操作。

<!-- more -->
# 1. 处理历史记录
## 1.1 查看
以下命令用来取得项目之前所有提交的列表

    git log

当然，可以用很多参数来精确控制历史记录的显示方式，这里推荐一种格式输出（参数意义自行查阅）

    git log --pretty=format:"%h %ad | %s%d [%an]" --graph --date=short
    # 输出例子：
    # d14ff74 2013-09-12 | First Commit (HEAD, master) [水月轻枫]

但是每次都要输入这一长串命令太不现实了，所幸Git提供别名功能，可以简化这个命令。

## 1.2 别名（alias）
`git status`，`git add`，`git commit`和`git checkout`这些命令太常用了，包括上面提到那个格式化的`git log`，缩写一下会大大提高效率。

要使用别名，把下面的内容加到你$HOME目录下的.gitconfig文件即可（Windows下的$HOME目录一般是“C:\Users\你的Windows用户名\”）。

    [alias]
        co = checkout
        cm = commit
        st = status
        br = branch
        hist = log --pretty=format:\"%h %ad | %s%d [%an]\" --graph --date=short

接下来，你就可以使用`git cm`来代替`git commit`，使用`git hist`来输出格式化的历史记录等等，方便吧。

## 1.3 获取旧版本
如果你已经配置了上述的查看历史命令的别名`git hist`，就可以很方便地使用这个命令来查看历史提交的列表。注意到每个提交的第一项都是一串简写的SHA1 hash值（完整的hash值可以说是一次提交的唯一标识），可以用以下命令来获取旧版本

    # 此处<hash>只需要前几个字符即可，如d14ff74
    git checkout <hash>

成功运行之后你的repo应该就处于对应的旧版本了。如果你需要返回主分支的最新版本，可以运行以下命令

    git checkout master

“master”是主分支默认的名字，如果checkout的是一个分支的名字，就会得到这个分支的最新版本。

## 1.4 给版本打标签
使用以下命令

    git tag v1

可以给当前版本打上标签“v1”。如果需要给过去的版本打标签，应先`git checkout`出以前的版本再打标签。

_小贴士：使用`git checkout v1^`可以签出v1的**上一个**版本，符号“^”表示“上一个版本”_

使用以下命令可以查看有哪些可用的标签

    git tag

使用`git hist master --all`可以在历史记录中看到标签信息。

# 2. 处理错误
## 2.1 在提交之前撤销更改
这里假定一个场景：你刚提交了一个新版本，然后对你的文件hello.py做了一些更改，随后你对刚才的更改不满意了，想将其恢复到提交时状态。这里分为两种情况：

* 本地修改（你还没用add或stage跟踪修改）：用`git checkout hello.py`即可恢复该文件，可见git checkout可以还原特定的文件。

* 已跟踪修改：用`git reset HEAD hello.py`把跟踪区重置到和HEAD版本一致的状态。注意：**reset命令（默认）不修改当前工作目录，所以工作目录里依旧会保留你的更改。**可以用git checkout命令来撤销工作目录中你不想要的修改。**这里需要你头脑很清晰地分辨跟踪区和工作目录的区别。**

## 2.2 撤销已提交的更改
有时你会突然意识到你刚刚提交的修改有问题，希望撤销这次提交。有好几种方法可以做到这一点，其中一种方法就是，就是再提交一次撤销了之前修改的新修改。

    # --no-edit选项指定不打开编辑器
    # 只要把HEAD换成其他hash值就可以恢复对应的版本
    git revert HEAD --no-edit

执行之后用git hist看一下记录，会发现有一次新的提交，这次提交覆盖了上一次的提交的内容。

上面这种方法虽然可以撤销更改，但是错误提交还是出现在git log里，这对于有强迫症的程序猿来说是多么不能忍！下面这个方法可以让这次错误的提交看起来像是完全没发生。

假定git hist输出以下内容

    # Oops是错误提交，Revert是用revert命令撤销的提交，注意到错误提交之前的版本带有v1标签
    $ git hist
    # 输出：
    # ee1237e 2011-09-20 | Revert "Oops, we didn't want this commit" (HEAD, master) [Jim Weirich]
    # 8021312 2011-09-20 | Oops, we didn't want this commit [Jim Weirich]
    # 1b303b2 2011-09-20 | Added a comment (v1) [Jim Weirich]

首先用`git tag oops`标记最后的提交，然后

    # --hard表示将当前工作目录更新到和新分支的最新版本一致
    git reset --hard v1

这时再查看历史记录，你会发现错误提交（Oops...）和撤销提交（Revert...）已经“消失”了。当然，**错误的提交并没有消失，它们还在库里。只是它们现在不会列在主分支里了，如果使用`git hist --all`还是可以看到记录。**

最后，不要忘了删除oops那个标签，以便**它所引用的提交可以被垃圾收集机制清理掉**

    git tag -d oops
    # 现在你用git hist --all也看不到了

## 2.3 修正提交
假设你刚刚用`git commit -m "No comment"`做了一次提交，然后马上意识到你在某个地方少添加了一句注释，但你确实不想为了这点屁大的事再做一次单独的提交，因此可以使用“修正提交”的方法。先加入你的注释，然后

    # --amend允许你修正最近一次提交
    # 不要忘记提交前git stage一下
    git commit --amend -m "Now add a comment"

再查看一下历史记录，你会发现“No comment”那次提交不见了，取而代之的是“Now add a comment”这次提交。