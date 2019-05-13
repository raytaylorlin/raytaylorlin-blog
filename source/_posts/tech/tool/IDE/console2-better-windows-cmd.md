title: Console2：Windows命令行威力加强版
date: 2014-05-15 15:36:29
categories:
- 技术
- 工具
- IDE
tags:
- Shell
- 控制台
---
作为一个Windows重度用户+程序猿，日常开发中免不了要经常使用命令行工具。但是Windows下默认的cmd提供的功能实在有限。今天无意间发现了一款很不错的命令行工具前端Console2，瞬间就被其深深地吸引，赶紧记下来分享一下。本文将简单介绍Console2及其配置方法，让你可以快速地配置出一个类似Linux终端的装逼利器。

<!-- more -->

和Linux下有强大的Konsole和gnome terminal不同，Windows里的命令行工具默认是cmd.exe，其简陋程度大家都清楚。当然Windows下还有一个Powershell，但在各方面也是差强人意。Console2这个工具是cmd.exe的前端包装，它有很多突出的优点：

* 支持多标签，这样就不会再任务栏里产生过多的窗口
* 不仅可以包装cmd.exe，也可以包装其他terminal程序，如cygwin、Powershell等等
* 支持透明背景，可配置字体和颜色，可配置热键等等
* 完全绿色，解压即用

官方下载地址：[http://sourceforge.net/projects/console/](http://sourceforge.net/projects/console/)

原版下载下来之后还需要经过一番配置才能成为装逼利器。

# 1. 基本配置

菜单栏`Edit | Settings`进入设置。以下列举我觉得比较重点的设置，其他的就自己摸索吧。

* Console: Shell：要包装的终端工具（默认为cmd.exe），用Powershell可以设置为`%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe`
* Console: Startup dir：默认启动的目录
* Console: Window size：启动时窗口的大小（也可以正常拖拽窗口右下角来调整）
* Console: Buffer size：缓冲区的大小（调大一点可以保留更多的内容）
* Apperance: 设置命令行工具的标题、字体、位置等（我设置为字体Consolas Size 15）
* Apperance|More: Window transparency：设置窗口激活和未激活时的窗口透明度（30, 80为佳）
* Behavior: Copy on select：选中文字时自动复制
* Hotkeys：设置快捷键。有一些非常好用的快捷键，比如`New Tab 1`设置为CTRL+T可以快速打开新标签，`Scroll buffer page up`设置为Page Up可以用键盘滚动命令行窗口，`Activate Console(global)`设置为CTRL+Shift+T可以让你在失去焦点的时候快速呼出命令行窗口，等等，各人按照各自的口味设置吧。**记住设置快捷键时要点Assign才能生效**
* Mouse：鼠标快捷键：`Select Text`设置为Left可以让你像选文本一样选择文字（配合上面的Copy on select可以让你选中文字马上就复制），`Paste text`设置为Right可以点击鼠标右键就粘贴
* Tabs：配置标签页。这里你可以配置默认命令行窗口的标题、图标、启动程序等等。我点击Add添加了一个Python Shell程序，然后再Hotkeys设置New Tab 2的快捷键，就可以一键启动python了！

# 2. 中文显示的问题

默认设置下中文字体会错位，让人很不爽。解决办法是菜单栏`View | Console window`调出原命令行窗口，然后在cmd的标题栏上`右键 | 属性 -> 字体 -> 新宋体`，然后确定保存。重启Console2，你会发现不会有屏幕文字偏移的问题了。

# 3. 无法中文输入的问题

网上很多博客都说无法输入中文的问题暂时无解，只能通过菜单栏`View | Console window`在原生的命令行窗口中输入中文。但经过一番搜索还是有比较完美的解决方案的。Github上有好心人对Console2的源码进行了修改并编译了一份可以支持中文输入的版本，猛戳[我Fork的版本](https://github.com/raytaylorlin/Console2-Chinese-Input-Capable)将`Console.exe`下载覆盖即可。

# 4. 通过右键菜单在当前目录打开命令行窗口

在Windows7中，在任意目录下Shift+鼠标右键，在菜单中可以看到“在此处打开命令行窗口”，点击后可以在当前目录打开cmd.exe。如果想在当前目录打开Console2，需要修改注册表。如果你在我的Github上下载了可以输入中文的Console2，应该会发现里面有两个bat批处理文件，一个是替换掉cmd，另一个是还原，只要运行一下bat即可修改右键菜单。**注意这两个文件要放在Console2目录下运行。**

最后放出一张示例图，看看这逼格满满的命令行窗口。

![Console2示例图](http://raytaylorlin-blog.qiniudn.com/image/IDE/Console2%E7%A4%BA%E4%BE%8B%E5%9B%BE.jpg)

