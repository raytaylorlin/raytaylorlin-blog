title: Django学习笔记（3）——Django的模板
date: 2014-2-9 18:07:55
categories:
- 技术
- 编程语言
- Python
tags:
- Django
- Python
---
在[学习笔记（1）](/Tech/Script/Python/django-note-1/)中，你可能已经注意到我们在例子视图hello_view中返回文本的方式有点特别。也就是说，HTML可以直接硬编码在Python代码之中。但是直接将HTML编码到Python代码中有许多坏处：对页面设计进行的任何改变都必须对Python代码进行相应的修改，站点设计的修改往往比底层Python代码的修改要频繁得多；Python代码编写和HTML设计是两项不同的工作，大多数专业的网站开发环境都将他们分配给不同的人员（甚至不同部门）来完成。

基于这些原因，将页面的设计和Python的代码分离开会更干净简洁更容易维护。我们可以使用Django的模板系统(Template System)来实现这种模式，这就是本文要具体讨论的问题。

<!-- more -->

# 1. Django模板系统

## 1.1 模板系统基础

模板是一个文本，用于分离文档的表现形式和内容。模板定义了占位符以及各种用于规范文档该如何显示的各部分基本逻辑（模板标签）。模板通常用于产生HTML，但是Django的模板也能产生任何基于文本格式的文档。下面这个模板例子描述了一个向某个公司签单人员致谢HTML页面。

{% raw %}
    <html>
    <head>
        <title>Ordering notice</title>
    </head>

    <body>
        <h1>Ordering notice</h1>
        <p>Dear {{ person_name }},</p>
        <p>Thanks for placing an order from {{ order.company }}. It's scheduled to
        ship on {{ order.ship_date|date:"F j, Y" }}.</p>
        <p>Here are the items you've ordered:</p>
        
        <ul>
        {% for item in item_list %}
            <li>{{ item }}</li>
        {% endfor %}
        </ul>

        {% if ordered_warranty %}
            <p>Your warranty information will be included in the packaging.</p>
        {% else %}
            <p>You didn't order a warranty, so you're on your own when
            the products inevitably stop working.</p>
        {% endif %}

        <p>Sincerely,<br />{{ company }}</p>
    </body>
    </html>
{% endraw %}

可以看出，模板跟HTML很像，事实上它就是保存成一个.html文件，它跟我们以前见到的HTML的区别就在于多了一些由双花括号`{%raw %}{{ }}{% endraw %}`括起来的**变量**以及由`{% raw %}{% %} {% endraw %}`括起来的模板标签，此外变量还可以通过过滤器（`|`管道符）转换变量输出格式。

变量在这里起到了占位符的作用，一般它最终会被传入模板的数据替代，最后模板将变成真正的HTML代码展现给用户。模板标签不会输出到最终的HTML中，它们多数起到了逻辑控制的作用，例如上例中，如果传入的ordered_warranty值为True，则显示相应的p标签；标签`{% raw %}{% for %}{% endraw %}`遍历传入的item_list，并依次生成结构相同但内容不同的HTML。

通常来说，我们会将模板和视图一起使用，但模板本身是一个Python库，你可以在任何地方使用它，而不仅仅是在Django视图中。这里简单介绍一下单独在Python代码中使用Django模板的最基本方式：

1. 可以用原始的模板代码字符串创建一个Template对象，Django同样支持用指定模板文件路径的方式来创建Template对象
2. 调用模板对象的render方法，并且传入一套变量context。它将返回一个基于模板的展现字符串，模板中的变量和标签会被context值替换。示例代码如下：

{% raw %}
    from django import template
    t = template.Template('My name is {{ name }}.')
    c = template.Context({'name': 'Adrian'})
    print t.render(c)
{% endraw %}

但实际的Django开发中我们一般不使用这种方式来渲染模板，后面将会看到更简单直接的方法。

## 1.2 模板标签介绍

1. `{% raw %}{% if %} {% endif %}{% endraw %}`标签检查一个变量，如果这个变量为真（即变量存在，非空，不是布尔值假），系统会显示在`if`和`endif`之间的任何内容。此外，还有一个可选的`else`标签，但是**没有**`elif`标签，所以如果需要多个分支条件，可以使用嵌套的`if`标签来代替。`if`标签可以接受`and`，`or`或`not`关键字来对多个变量做判断，并且**不允许在同一个标签中同时使用and和or，因为逻辑上可能模糊的**。
2.  `{% raw %}{% for %} {% endfor %}{% endraw %}`允许我们在一个序列上迭代，每一次循环中，模板系统会渲染在`for`和`endfor`之间的所有内容。每个for循环里还有一个称为`forloop`的模板变量，这个变量有一些提示循环进度信息的属性，关于这个变量的详细用法可以参照[这篇文章](http://djangobook.py3k.cn/appendixF/)的for一节。下面一段示例代码给出和for标签结合使用的reverse（反向遍历）和empty（列表为空的情况）标签的用法：

    {% raw %}
    {% for athlete in athlete_list reverse%}
    <p>{{ athlete.name }}</p>
    {% empty %}
    <p>There are no athletes. Only computer programmers.</p>
    {% endfor %}
    {% endraw %}

3. `{% raw %}{% ifequal VARIABLE CONSTANT %} {% endifequal %}{% endraw %}`标签比较VARIABLE和CONSTANT两个值，当他们相等时，显示在`ifequal` 和`endifequal`之中所有的值
4. `{% raw %}{# #}{% endraw %}`是Django模板语言的代码注释
5. 就像前面提到的一样，模板过滤器是在变量被显示前修改它的值的一个简单方法，而且一个过滤器管道的输出又可以作为下一个管道的输入。常用的过滤器有`first`，`upper`，`truncatewords`等等，详情同样参照[这篇文章](http://djangobook.py3k.cn/appendixF/)。

# 2. 在视图中使用模板

## 2.1 配置模板路径

为了减少模板加载调用过程及模板本身的冗余代码，Django提供了一种使用方便且功能强大的API，用于从磁盘中加载模板，要使用此模板加载API，首先你必须将模板的保存位置告诉框架。假设项目根目录是mysite，一般会在这个目录下建立一个templates文件夹，专门存放模板。然后在setting.py中设置`TEMPLATE_DIRS`这一项。

    TEMPLATE_DIRS = (
        'templates/',
    )

## 2.2 渲染模板

Django提供了一个捷径，让你一次性地载入某个模板文件，渲染它，然后将此作为HttpResponse返回。我们以下面这样一个模板文件（current_datetime.html）为例。

{% raw %}
    <html><body>It is now {{ current_date }}.</body></html>
{% endraw %}

那么现在的视图views.py可以这样写。

    from django.shortcuts import render_to_response
    import datetime

    def current_datetime_view(request):
        # 获取填充模板的变量，这里经常会是调用Model层的方法访问数据库
        now = datetime.datetime.now()
        # 第一个参数是要使用的模板名称，第二个参数是一个字典，用于填充模板
        return render_to_response('current_datetime.html', {'current_date': now})

以上模板加载、上下文创建、模板解析和HttpResponse创建工作均在对均被`render_to_response()`的调用给包办了了，因为这个函数返回了HttpResponse对象，所以我们仅需在视图中 return该值。

# 3. 模板的编写原则

在介绍了模板加载机制之后，我们再介绍一个利用该机制的内建模板标签：`{% raw %}{% include %}{% endraw %}` 。该标签允许在（模板中）包含其它的模板的内容。标签的参数是所要包含的模板名称，可以是一个变量，也可以是用单/双引号硬编码的字符串。每当在多个模板中出现相同的代码时，就应该考虑是否要使用`include`来减少重复，这也是为了提高代码的可重用性。

在实际应用中，经常会用Django模板系统来创建整个HTML页面。这就带来一个常见的 Web 开发问题：在整个网站中，如何减少共用页面区域（比如站点导航）所引起的重复和冗余代码？

解决该问题的传统做法是使用服务器端的includes，你可以先编写各小部分（比如导航栏、header、footer、各功能区块等等），然后在一个base.html中搭出网页的HTML骨架，然后用`include`像拼拼图一样各个模板组装成最后的大页面。但是用Django解决此类问题的首选方法是使用更加优雅的策略——*模板继承*。你可以对那些**不同**的代码段进行定义，而不是定义**相同**的代码段。

第一步是定义*基础模板base.html*，该模板之后将由*子模板*继承。

{% raw %}
    {% load staticfiles %}
    <!DOCTYPE html>
    <head>
        <title>
            {% block title %} The Default Title {% endblock %}
        </title>

        {% block css %} 
        <link href="{% static "css/base.css" %}" rel="stylesheet" type="text/css"/>
        {% endblock %}
    </head>
    <body>
        <div id="wrapper">
            {% block header %}
            {% include 'partial/header.html' %}
            {% endblock %}

            {% block content %} {% endblock %}

            {% block footer %}
            {% include 'partial/footer.html' %}
            {% endblock %}
        </div>

        {% block js %} 
        <script type="text/javascript" src="{% static "js/jquery.min.js" %}"></script>
        {% endblock %}
    </body>
{% endraw %}

所有的`block`标签告诉模板引擎，子模板可以重载这些部分，如果子模板不重载这些部分，则将按默认的内容显示。（顺便提一下，模板中的`load staticfiles`表示加载静态资源，这个一般用于加载CSS,JS等静态文件时用到）上面的基础模板中定义了6个block，当我们的页面有变化时，随时可以对这些block做出修改，看看下面的首页index.html：

{% raw %}
    {% extends "base.html" %}
    {% load staticfiles %}

    {% block css %} 
    {{ block.super }}
    <link href="{% static "css/page/index.css" %}" rel="stylesheet" type="text/css"/>
    {% endblock %}

    {% block content %}
    <p>Hello world!</p>
    {% include "partial/index/product_list.html" %}
    {% endblock %}
{% endraw %}

在index.html中，首先用`extends`标签声明这个模板继承的父模板（必须保证其为模板中的第一个模板标记）。接着重定义css这个block，在包含了父模板的基础上（`block.super`）引入了新的CSS文件（也就是说，这里最终会加载base.css和index.css两个文件）；替换了content这个block。这就是子模板所做的全部工作。注意：**继承并不会影响到模板的上下文。换句话说，任何处在继承树上的模板都可以访问到你传到模板中的每一个模板变量。**

最后总结一下Django模板的编写原则。你可以根据需要使用任意多的继承次数。使用继承的一种常见方式是下面的三层法：

1. 创建base.html模板，在其中定义站点的主要外观感受。这些都是不常修改甚至从不修改的部分。
2. 为网站的每个区域创建base_SECTION.html模板(例如, base_photos.html和base_forum.html)。这些模板对base.html进行拓展，并包含区域特定的风格与设计。
3. 为每种类型的页面创建独立的模板，例如论坛页面或者图片库。这些模板拓展相应的区域模板。

这个方法可最大限度地重用代码，并使得向公共区域（如区域级的导航）添加内容成为一件轻松的工作。