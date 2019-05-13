title: IntelliJ IDEA 12.0中文显示问题解决方案
date: 2013-04-2 15:08:29
categories:
- 技术
- 工具
- IDE
tags:
- IDE
- IntelliJ IDEA
---
IntelliJ IDEA是一个非常强大的IDE，但是只有英文版，且默认的中文显示有一定问题。本文介绍了IntelliJ IDEA 12.0中文显示问题解决方案。

<!-- more -->

### 1. IDE本身的中文乱码
这个问题体现在IDE本身，比如打开文件浏览目录的时候，中文名的文件或目录会显示成方块。

解决方法：  
1. 进入设置页。File->Settings。  
2. 进入IDE Settings里的File Encodings项，把IDE Encoding项设置成UTF-8。确定。  
3. 进入IDE Settings里的Appearance项，选中Override default fonts by，把Name设置为你喜欢的字体（我使用的是Yahei Consolas Hybrid），Size根据自己喜好设置（我一般设为 14）。确定。

以上应该可以保证中文显示没有问题了。

### 2. 编辑器的中文问题
这个问题体现在代码编辑区中写中文时，可能会乱码或者中文汉子全部重叠在一起。

首先要确定你正在编辑的文件是UTF-8编码的，有很多文件可能默认是ANSI编码。

至于中文重叠那是因为你所选用的默认中文字体不对，一直以来我写代码都是用的consolas，但是这个字体不支持中文，Intellij IDEA 12中如果使用默认的中文字体（不知道是哪个字体）就会重叠在一起，在网上找了好久，终于找到一个神一般的字体*Yahei Consolas Hybrid*，即微软雅黑和consolas的混合！

于是乎，File->Settings IDE Settings->Editor->Color & Fonts->Font，设置字体为Yahei Consolas Hybrid即可。

**[神一般的字体Yahei Consolas Hybrid下载](http://pan.baidu.com/s/1c0lAVfE)**

如果发现安装了这个字体但是在设置中找不到的话，尝试使用以下这个方法：  
按图所示，保存为另外一个名字，由你喜欢。最好是英文字母组成，这里我们保存为Darcula1。

![Intellij IDEA 12 设置字体](http://raytaylorlin-blog.qiniudn.com/image/IDE/Intellij%20IDEA%2012%20%E8%AE%BE%E7%BD%AE%E5%AD%97%E4%BD%93.jpg)  

在此路径（win7）“C:\Users\你的计算机名\\.IntelliJIdea12\config\colors”找到Darcula1.xml文件。  
用记事本打开Darcula1.xml文件，把第8行  

    <option name="EDITOR_FONT_NAME" value="你之前保存的字体" />

改为  

    <option name="EDITOR_FONT_NAME" value="YaHei Consolas Hybrid" />

然后重启IntelliJ IDEA 12.0，中文字符问题解决，效果如下图。

![Intellij IDEA 12中文字符显示效果](http://raytaylorlin-blog.qiniudn.com/image/IDE/Intellij%20IDEA%2012%E4%B8%AD%E6%96%87%E5%AD%97%E7%AC%A6%E6%98%BE%E7%A4%BA%E6%95%88%E6%9E%9C.jpg)

