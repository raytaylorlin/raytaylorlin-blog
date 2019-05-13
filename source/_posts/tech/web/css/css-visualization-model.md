title: CSS学习笔记：可视化格式模型
date: 2014-08-02 21:21:34
categories:
- 技术
- Web前端
- CSS
tags:
- CSS
- 盒模型
---
浮动、定位和盒模型是CSS三个最重要的基础概念，也是非常容易混淆和难掌握的概念。这些概念用于控制在页面上摆放和显示元素的方式，形成CSS基本布局。本文将介绍盒模型概念，外边距叠加，绝对定位和相对定位的差异，以及浮动和清理如何工作等。

<!-- more -->

# 1. 盒模型

## 1.1 概述

CSS盒模型指定元素如何显示以及在某种程度上如何相互交互。页面上的每个元素都被看做是一个矩形框，这个框由元素的内容、内边距、边框和外边距组成，如下图所示。如果使用Chrome的DevTools审查元素功能，鼠标指向页面随意一个元素，可以看到蓝色部分是内容，绿色部分是内边距，橙色部分是外边距。CSS中，width和height指的是内容区域的宽度和高度，增加padding、border、margin都不会影响内容区域的尺寸，但是**会增加元素框的总尺寸**。

![CSS标准盒模型](http://raytaylorlin-blog.qiniudn.com/image/css/CSS%E6%A0%87%E5%87%86%E7%9B%92%E6%A8%A1%E5%9E%8B.jpg)

* 内边距padding：如果在元素上添加背景（`background`属性），背景会应用于内容和内边距组成的区域。
* 边框border：在内边距的区域外边加一条线，可以有实线、虚线、点线等。
* 外边距：透明，一般用于控制元素之间的间隔。

## 1.2 IE的盒模型

IE的早期版本（包括IE6）在混杂模式中使用自己**不符合标准**的盒模型，即这些浏览器的width属性不是内容的宽度，而是内容、内边距和边框的宽度总和。这种不标准的设定会引起很多麻烦，也是早期前端开发人员极其厌恶IE的原因。尽管它是不标准的，但是了解IE的盒模型对于建立兼容老式浏览器的网站还是很有帮助的。（CSS3的box-sizing属性可以定义要使用哪种盒模型）

## 1.3 外边距叠加

当两个或更多**垂直**外边距相遇时，它们的实际效果只形成**一个**外边距，其高度等于多个发生叠加外边距的最大者。例如，当两个元素垂直排列，上面元素的margin-bottom会合下面元素的margin-top发生叠加。外边距叠加实际还有很多种情形，包括空元素的外边距叠加等等，详情及外边距合并的意义参见[W3School CSS外边距叠加](http://www.w3school.com.cn/css/css_margin_collapsing.asp)。注意：只有普通文档流中块框的垂直外边距才会发生外边距合并，行内框、浮动框或绝对定位之间的外边距不会合并。

# 2. CSS定位机制

CSS有三种基本的定位机制：普通流、绝对定位和浮动。默认为普通流，即元素框的位置由元素在HTML中的位置决定。

## 2.1 块和行内元素

p、h1、div等元素常常称为块级元素（块框），它们显示为一块内容；i、span等元素称为行内元素（行内框），它们的内容显示在一行中。可以使用CSS的display属性改变生成框的类型，例如给span设置`display:block`可以让其行为变得和块框一样。块框从上到下垂直排列，行内框在一行中水平排列，默认垂直内外边距和边框不影响行内框的高度，不过如果使用`display:inline-block`可以让元素既像行内元素一样水平排列，又符合块级框的行为。

如果将一些文本添加到一个块级元素（如div）的开头时，这些文字会被当做块级元素处理，这个块框称为匿名块框，而且无法直接应用样式。如下面代码中的“some text”就是这种文字。

    <div>
        some text
        <p>more</p>
    </div>
    
## 2.2 相对定位和绝对定位
    
相对定位概念参见[W3School CSS相对定位](http://www.w3school.com.cn/css/css_positioning_relative.asp)，绝对定位概念参见[W3School CSS绝对定位](http://www.w3school.com.cn/css/css_positioning_absolute.asp)。基本概念此处不再赘述。

要注意相对定位的元素被看做普通流的一部分，但**绝对定位会使元素脱离文档流，不占据空间**。并且绝对定位元素的位置是相对于最近的已定位的祖先元素的，通常使用`position:absolute`的元素会被包含在`position:relative`的父元素之中。

## 2.3 浮动

浮动概念参见[W3School CSS浮动](http://www.w3school.com.cn/css/css_positioning_floating.asp)。

浮动的元素会脱离文档流（不占据空间），为了使父元素在视觉上包围浮动元素，W3School中给出的示例介绍了一种使用一个空元素来清理浮动的方法：`<div class="clearfix"></div>`，`.clearfix {clear:both;}`。这个方法的缺点是会添加一个无意义的标签。

现在更常用的方法是在父容器上应用`.clearfix`类，HTML结构如下：

    <div class="news clearfix">
        <img src="/img/news.jpg" class="pull-left">
        <p class="pull-right">Some text</p>
    </div>

对应的clearfix类CSS代码：

    .clear:after {
        content: ".";
        height: 0;
        visibility: hidden;
        display: block;
        clear: both;
    }

这个方法结合使用`:after`伪类在父容器末尾添加一个不引人注意的字符“.”（其他字符也可以，因为height为0且visibility为hidden是看不到的），然后通过将display设为block使被清理的元素在它们的margin-top添加了空间。这样设置之后就可以对生成的内容进行清理，又避免了添加空元素。这个方法除了IE6意外，在多数现代浏览器中是有效的。

参考资料：精通CSS:高级Web标准解决方案（第2版）第三章