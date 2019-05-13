title: Git指南（1）——安装与基本用法
date: 2013-09-11 22:07:42
categories:
- 技术
- 工具
- Git
tags:
- Git
---
现在很多开发团队，包括我所在实验室，使用的版本控制系统都是SVN。Git作为一个新兴的快速的分布式版本控制系统，却很少有人知道或使用。我使用Git的时间不长，平时使用的命令也就那么几个，但深知Git是一个前途无量的东西，因此决定尝试着坚持使用并让更多的人知道它。

之前从来没有对Git进行过系统的学习，只了解其中的一点皮毛原理和使用方式。这次正好借这个机会完整地学习一下，并把本系列作为一个教程或学习笔记。如果你对Git有兴趣，这个系列可以作为入门的参考指南。（注：本系列均以Windows版本为例，Linux或Mac下的命令可能有点不同，请自行查阅；另外，本系列所讲的都是Git的命令行，如果你使用小乌龟客户端GitTortoise，那么只需要动几下鼠标就可以完成操作）

<!-- more -->
# 1. 为什么使用Git
关于Git的优缺点，与SVN的对比网上的资料有非常多，这里只列举几点比较重要的。

* 分布式版本控制，文件变更的历史记录不再存在一个集中服务器，如果服务器故障，或者没有网络，几乎所有操作都可以在本地执行，在任何地方都可以愉快地提交更改。
* 版本的分支（branch）和合并（merge）十分方便。
* 有一个全世界最大的开源社区——github，上面可以看到各种优秀的各种语言的代码，并且越来越多的项目由github托管。

# 2. 参考材料
* [Pro Git中文版](http://iissnan.com/progit/index.html)
* 一个极其简洁明了的Git教程网站[Git Immersion](http://gitimmersion.com/)
* [Git Community Book 中文版](http://gitbook.liuhui998.com/index.html)

# 3. 安装
安装Git Windows客户端（命令行界面）：[msysgit](http://msysgit.github.io/)

用SVN的人一定对小乌龟客户端很熟悉，Git也有对应的版本：[GitTortoise](https://code.google.com/p/tortoisegit/)

Linux或Mac用户请自寻谷歌安装方法。

# 4. 基本用法
## 3.1 配置
在使用Git之前，需要打开命令行客户端

1.设置名字和信箱

    git config --global user.name "你的名字"
    git config --global user.email "your_email@whatever.com"

2.设置换行符参数

    # 提交时转换为LF，检出时转换为CRLF
    git config --global core.autocrlf true
    # 拒绝提交包含混合换行符的文件
    git config --global core.safecrlf true
    
## 3.2 流程
在你的工程文件夹下，执行以下命令创建代码仓库（Repository，就是在网上经常看到的repo）

    git init

假设你的工程有一个index.html文件，以下命令将其添加到跟踪改动状态（stage）

    git add index.html
    # git add . 可以跟踪所有文件
    # 此处也可以使用git stage来跟踪改动

查看状态

    git status
    # 此时你应该看到输出提示add了index.html，其他状态还有modified（修改）等等

查看与上一次提交的差异（具体参考[这里](http://gitbook.liuhui998.com/3_5.html)）

    git diff
    # 加上--cached参数，git diff会显示当前你所有已跟踪的修改
    git diff branch-name    # 查看当前工作目录与指定分支的差别

然后提交

    # -a表示跟踪所有文件的改动（相当于先做了一次git add .，再commit），-m 后面跟着提交的说明
    git commit -a -m "First Commit"

## 3.3 需要特别注意的问题

**每一次你对文件做了改动（包括删除），都需要使用`git add`或`git stage`来跟踪改动，只有被跟踪改动了的文件才可以使用`git commit`提交。**

增加跟踪改动（stage）主要有两个好处，一个是分批、分阶段递交，一个是进行快照，便于回退。

1.分批提交：比如，你修改了a.py, b.py, c.py三个文件，其中a.py和c.py 是功能1的相关修改，b.py属于功能2的相关修改。那么你就可以采用以下方式分批提交。

    git stage a.py c.py
    git commit -m "function 1"
    git stage b.py
    git commit -m "function 2"

2.分阶段提交
比如你修改了a.py，然后`git stage a.py`，相当于对当前的a.py 做了一个快照，然后又做了一些修改，这时候，如果直接采用`git commit`递交，则只会对第一次的快照进行递交，当前内容还保存在工作区。需要再做一次 `git stage`，才能提交。

3.文件快照，便于回退
当文件加入了stage区以后，如果要从stage区删除，则使用 reset，此时工作区的文件不做任何修改，比如：
    # 这个命令就是git stage a.py的反操作
    git reset a.py

当文件加入了 stage 区以后，后来又做了一些修改，这时发现后面的修改有问题，想回退到stage的状态，使用checkout命令

    git checkout a.py

4.忽略某些文件

一般一个项目总会有些文件无需纳入Git的管理，比如一些自动生成的日志文件，或者编译过程中创建的临时文件等。我们可以创建一个名为`.gitignore`的文件，列出要忽略的文件模式。下面是一个例子：

    # 此为注释 – 将被 Git 忽略
    # 忽略所有 .a 结尾的文件
    *.a
    # 但 lib.a 除外
    !lib.a
    # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
    /TODO
    # 忽略 build/ 目录下的所有文件
    build/
    # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
    doc/*.txt

本篇就先讲到这里，下一篇将会讲Git的历史操作。
