title: 使用Markdown来写毕业论文
date: 2015-12-17 16:17:53
categories:
- 技术
- 综合
- 项目
tags:
- Markdown
- 论文
---
最近很长一段时间没有写博客，主要是因为“学”的时间减少了，但私底下“做”的时间增加了。作为研三的老人，马上就要进入写毕业论文的阶段了，所以最近一直都在准备论文的事情，所所以博客的更新也会稍微缓一缓，估计到放假回家就可以恢复正常了。

说到准备论文，最近一直都在研究如何使用Markdown来写毕业论文，而本文就是研究成果[hust-graduation-thesis-pandoc](https://github.com/raytaylorlin/hust-graduation-thesis-pandoc)的展示。项目现已放在Github上，欢迎各位使用和反馈。

<!-- more -->

实验室以前的学长学姐写毕业论文，都是规规矩矩地在前人的Word文档基础上修改。但是作为一个程序猿，用Word来写论文未免太不够档（zhuang）次（bi）。更重要的是用Word来写论文时，**经常在格式上会出现一些莫名其妙的问题**，比如保存时明明是好的下次打开时多级标题全错乱了啊，正文的字体有时会突然错乱了啊等等，而且无论怎么调都调不好，最后只能在那气得干瞪眼，然后重新找一份前人“及格”的文档重新编辑。

当然，科研界的投稿，很多都是用[Latex](https://zh.wikipedia.org/wiki/LaTeX)来编写。Latex虽然超级好用，但是学习曲线太陡峭了，并不适合我们这种在一个月内就要写完论文的人士。当然，经过考验的格式编排没有问题的Latex模板网上有很多，一般拿过来直接往里面填充内容就可以了。不管怎么说，除去折腾Latex的安装和模板的制作，直接站在前人的肩膀上使用Latex模板绝对要比使用Word模板靠谱得多。

但是Latex本身是“内容和样式”混排的，[Markdown](https://zh.wikipedia.org/wiki/Markdown)这种简洁优雅只专注于内容的标记语言才是程序猿的最爱。至于用Markdown来写作的优势，此处就不赘述了，懂的人应该都懂的。而最近的研究成果，就是实现了一套方案，既能让人利用Markdown的语法优势快速编写内容，又能利用Latex的强大和稳定，生成漂亮的pdf论文。要注意的是，原生的Markdown提供的功能并不足以胜任论文的编写，因为论文可能会包含公式、表格、交叉引用等等。幸运的是，使用[Pandoc Markdown](http://pandoc.org/README.html#pandocs-markdown)这种增强版的Markdown语法，再混编一点点Latex指令，就可以解决我们的问题。

事实上，早在几年前我的好基友[@pyrocat101](https://github.com/pyrocat101)就已经用Markdown完成了他的本科毕业论文。而我的这个项目也是参考了他的项目[hust-thesis-pandoc](https://github.com/Sicun/hust-thesis-pandoc)，同时也略微修改了同系同学[@xu-cheng](https://github.com/xu-cheng)提供的[华中科技大学毕业论文Latex模板](https://github.com/hust-latex/hustthesis)，加上自己的一套构建方法而成。所以我也要向这两位伟大的先驱者致敬。

项目的使用方法和注意事项，Github项目的文档已经写得很清楚了，这里就只简单说下我所做的工作。

1. @xu-cheng 提供的Latex模板适用于纯Latex用户，我运行了其提供的`makewin32.bat unpack`脚本，从[hustthesis.dtx](https://github.com/hust-latex/hustthesis/blob/master/hustthesis/hustthesis.dtx)模板中提取出了cls和bst文件，这两个才是编写latex时真正要用到的模板。
2. 用Markdown写一份论文示例，使用[Pandoc](http://pandoc.org/)这款工具，将md转换为tex。
3. 用[Tex Live](https://www.tug.org/texlive/)提供的`xelatex`工具（`lualatex`亦可），编译上述tex文件，生成最终的pdf文件。
4. 参考机油项目中的[Makefile](https://github.com/Sicun/hust-thesis-pandoc/blob/master/Makefile)，编写一份[Gulp](http://gulpjs.com/)构建脚本，将上述过程自动化。
5. 将latex模板中的个人信息、摘要、致谢等内容分离出单独的tex文件，并整理整个项目结构。

本项目目前仍处于试验期，我会在写毕业论文期间不断维护这个项目，欢迎各位评测指正。
