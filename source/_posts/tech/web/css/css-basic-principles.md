title: CSS学习笔记：基本原则
date: 2014-07-29 09:38:05
categories:
- 技术
- Web前端
- CSS
tags:
- CSS
---
本文记录一些CSS的基本的但常常被误解的概念，并讨论如何让HTML和CSS保持清晰且结构良好，内容包括设计代码的结构、有意义文档的重要性、命名约定、何时使用ID和类名、文档类型和浏览器模式等。

<!-- more -->

# 1. 基本原则

## 1.1 使用有意义的HTML标记

早期的Web仅仅使用HTML来表现页面，经常使用字体和粗体标签来创建所需的视觉效果，而不只是用标题元素突出标题；表格成了一种布局工具而不是显示数据的方式；使用块引用（blockquote）来添加空白而不是表示引用……早期的Frontpage和Dreamweaver等编辑器能够通过简单的拖拽操作构建复杂的表格布局，但是*嵌套的表格*和*分隔线GIF*把代码弄得非常混乱，而且这些布局极其脆弱，很容易被破坏。HTML没有被当做简单的标记语言，许多人害怕直接编写网页代码，宁可依赖可视化编辑器。

在Web一团糟的背景下，CSS的出现使得Web开发者可以控制页面的外观，并将文档的表现部分与内容分隔开。只需要在一个地方进行修改，就能贯彻到整个站点，而且可以使用CSS而不是表格来控制布局。HTML标记返璞归真，又变得有实际意义了。与*表现性*的页面相比，*有意义*的页面更容易处理。HTML包含了丰富的有意义的元素，如：`h1`、`h2`、`ul`、`blockquote`、`abbr`、`strong`、`code`、`label`、`tbody`等等。如果元素有恰当的含义，就应该使用。

## 1.2 ID和类名

有意义的标记可以提供良好的基础，但是可以用的元素还不够全面。HTML4是作为简单的文档标记语言创建的，而HTML5提供了更丰富的元素，如`header`、`nav`、`article`、`section`、`footer`等结构性元素。
次优的解决方案是使用现有的元素，如`div`、`ul`等等，然后通过添加ID或类名类赋予额外的意义，如`<ul id="nav"></ul>`。当然是用ID是有一定局限性的，实际上CSS的最佳实践并不推荐使用ID，甚至有的CSS检查器默认会对使用ID提出警告。如果使用大量ID，很快就会难以找到唯一的名称，最终不得不创建很长很复杂的命名约定。只有在绝对确定这个元素只会出现一次的情况下才使用ID，如果不确定的话，应该首选CSS类，毕竟类是比较灵活和可复用性较强的。

## 1.3 CSS类的命名和使用

在分配ID和类名时，一定要尽可能保持名称与表现方式无关，应该根据**它们是什么**来为元素命名，而不是根据**它们的外观如何**来命名。例如要让一个表单通知消息显示红色，不应该分配类名`.red`，而是`.notification`之类有意义的名字，这样可以在整个网站中重用它们，修改起样式来也非常方便。关于CSS类（ID）的命名，推荐采用完全小写及多个单词之间用连字符分隔，如`.btn-primary`。

由于CSS类功能强大，所以它们可能被滥用。CSS新手常常在几乎所有东西上添加类，试图更精细地控制样式，这种现象被称为“多类症”。HTML代码应该保持干净整洁，尽量避免无意义的标记和属性。看看下面一段表示新闻的代码，其中就有点患了“多类症”：

    <div class="news">
        <h2 class="news-head">...</h2>
        <p class="news-text">...</p>
        <p class="news-text">
            <a href="news.php" class="news-link">More</a>
        </p>
    </div>
    
上述示例用了太多的CSS类名来区分每个元素，实际可以使用层叠（cascade来识别新闻标题和文本：

    <div class="news">
        <h2>...</h2>
        <p>...</p>
        <p><a href="news.php">More</a></p>
    </div>
        
只要你发现类名中出现了重复的单词，就应该考虑是否可以把这些元素分解成它们的组成部分。以这种方式删除不必要的类有助于简化代码。如果你发现非得添加许多类才能解决问题，这很可能意味着你的HTML文档结构有问题。

## 1.4 div和span

现代的Web开发都流行使用div标签来排版和布局，但是许多人误以为div元素没有语义。实际上div代表部分（division），它可以将文档分割成几个有意义的区域。为了将不必要的标记减到最少，应该只在没有现有元素能实现区域分割的情况下才使用div。

过度使用div被称为“多div症”，这是代码结构不合理而且过分复杂的一个信号。例如前端新手可能会这样写一个主导航列表：`<div class="nav"><ul>...</ul></div>`，但这个div实际是完全没必要的，直接在ul上使用nav类。使用div应该根据条目的意义或功能，而不是根据表现方式或布局来进行分组。

div可以用来对块级元素分组，而span可以用来对行内元素分组。尽管目标是让代码尽可能简介或有意义，但是有时为了以自己希望的方式显示页面，无法完全避免添加额外的无语义的div或span，这样也不必过分为此担心。关键是要知道在什么时候进行折中。

# 2. 文档类型和浏览器模式

## 2.1 文档类型

DTD（文档类型定义）是一组机器可读的规则，它们定义XML或HTML的特定版本中允许和不允许有什么。浏览器解析网页时，将使用这些规则检查页面的有效性并采取相应的措施。DOCTYPE声明是指HTML文档开头处一两行描述使用哪个DTD的代码，浏览器通过DOCTYPE声明知道要使用HTML哪个版本。DOCTYPE有严格（strict）和过渡（transitional）两种风格，过渡DOCTYPE的目的是帮助开发人员从老版本迁移到新版本。
## 2.2 浏览器模式

浏览器厂商为了创建与标准兼容的浏览器并确保向后兼容，他们实现了标准模式和混杂模式（quirks mode）两种呈现模式。在标准模式中，浏览器根据规范呈现页面；在混杂模式中，页面以一种比较宽松的向后兼容的方式显示，比如IE6的混杂模式使用了老式的专有盒模型。

浏览器会根据DOCTYPE是否存在及使用哪种DTD来选择呈现模式。书写包含正确的DTD的DOCTYPE才能让页面以标准模式呈现，否则将以混杂模
式呈现。一些常用的DOCTYPE声明可以参照[HTML DOCTYPE Declaration](http://www.w3schools.com/tags/tag_doctype.asp)。

参考资料：精通CSS:高级Web标准解决方案（第2版）第一章