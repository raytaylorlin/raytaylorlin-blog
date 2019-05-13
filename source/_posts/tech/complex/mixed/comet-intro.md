title: Comet：基于 HTTP 长连接的“服务器推”技术简介
date: 2013-05-06 18:13:53
categories:
- 技术
- 综合
- 杂项
tags:
- Comet
---
最近实验室生物信息Heatmap项目正在考虑做一个这样的功能：一个用户在heatmap（一张超大的可能是GB级别大小的图片）上做一个标注，另一个在线的用户可以马上就看得出来，相当于在线协作。其实这功能的原理就有点类似现在很多网站上的即时通讯，比如Web QQ和网页版的阿里旺旺。按照学长的指示，我开始接触到一种叫Comet的技术，也就是本文所要汇总的内容。

<!-- more -->

关于Comet的入门介绍，这里有一篇很好的[文章](http://www.ibm.com/developerworks/cn/web/wa-lo-comet/)。

接下来是5篇很详细的博文，从Comet各种实现方式的介绍，到WebSocket，到各种常用的Comet框架介绍都有。

1. [反向Ajax第1部分——Comet介绍][comet1]
2. [反向Ajax第2部分——WebSocket][comet2]
3. [反向Ajax第3部分——Web服务器和Socket.IO][comet3]
4. [反向Ajax第4部分——Atmosphere和CometD][comet4]
5. [反向Ajax第5部分——事件驱动的Web开发][comet5]

[comet1]: http://blog.csdn.net/coderjiang/article/details/8675120
[comet2]: http://blog.csdn.net/coderjiang/article/details/8675539
[comet3]: http://blog.csdn.net/coderjiang/article/details/8676823
[comet4]: http://blog.csdn.net/coderjiang/article/details/8676882
[comet5]: http://blog.csdn.net/coderjiang/article/details/8676923

以上如果想要快速了解概况以及各种技术的对比，看第1部分就够了（包括后面几部分其中涉及的Java代码基本上不用看）。第2部分涉及较新的HTML5的WebSocket也是非常强大，值得关注。第3部分介绍了Socket.IO这个很强大的js库，简单好用，也是我初步决定采用的即时通讯技术方案。最后两部分还没细看，目测也用不着。

看了几天的文章之后，初步对Comet技术有了大体的认识，也了解到目前使用最多的实现方式应该是采用ajax长轮询，而较新且更强大的是使用WebSocket。而在丁基友的推荐下，我可是接触Socket.IO这个东西，貌似可以兼容大部分常见浏览器，并自动采用最佳方式来满足这种即时通讯的需要，于是决定了下一步开始对Socket.IO展开研究。
