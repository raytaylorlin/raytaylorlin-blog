title: Django学习笔记（1）——开始Django
date: 2014-1-27 11:31:24
categories:
- 技术
- 编程语言
- Python
tags:
- Django
- Python
---
最近需要对某个产品展示网站的数据库做一些修改，之前也听说过各种python的Web框架，正好可以拿来学习和练练手。貌似对初学者来说，django是一个比较老牌而且功能也很强大的框架，虽然它有很多缺点，但还是决定先从这个框架入手学习。本系列是一个简单的学习笔记，供以后参考。

网站的后台原来是用PHP的CodeIgniter框架写的，数据库只有4个表，涉及最基本的产品的增删查改和切图等操作。现在换用django后，所有后台包括模板都得重写。虽然看起来工作量很大，但实际做起来也就花三天学了下django，然后再用三天把所有后台重写（不包括部署到蛋疼的SAE的繁琐过程），可见django的开发效率是如此之高。

<!-- more -->

# 1. 引言
## 1.1 简介

Django是一个由Python写成的，开源的并采用了MVC设计模式的Web应用框架。但是在Django中，控制器接受用户输入的部分由框架自行处理，所以Django里更关注的是模型（Model）、模板(Template)和视图（Views），称为MTV模式。它们各自的职责如下：
* 模型（Model），数据存取层：处理与数据相关的所有事务，即如何存取、如何验证有效性、包含哪些行为以及数据之间的关系等。
* 模板(Template)，表现层：处理与表现相关的决定，即如何在页面或其他类型文档中进行显示。
* 视图（View），业务逻辑层：存取模型及调取恰当模板的相关逻辑。模型与模板之间的桥梁。

Django的MVC编程方式不同于CodeIgniter那样把业务逻辑都放在控制器中，Django的MVC控制器部分由URLconf来实现。URLconf机制是使用正则表达式匹配URL，然后调用合适的Python函数。它比MVC框架考虑的问题要深一步，因为我们程序员大都写程序在控制层。现在这个工作交给了框架，仅需写很少的调用代码，大大提高了工作效率。

## 1.2 特性

Django框架的核心包括：一个面向对象的映射器，用作数据模型（以Python类的形式定义）和关联性数据库间的媒介；一个基于正则表达式的URL分发器；一个视图系统，用于处理请求；以及一个模板系统。核心框架中还包括：
* 一个轻量级的、独立的Web服务器，用于开发和测试。
* 一个表单序列化及验证系统，用于HTML表单和适于数据库存储的数据之间的转换。
* 一个缓存框架，并有几种缓存方式可供选择。
* 中间件支持，允许对请求处理的各个阶段进行干涉。
* 内置的分发系统允许应用程序中的组件采用预定义的信号进行相互间的通信。
* 一个序列化系统，能够生成或读取采用XML或JSON表示的Django模型实例。
* 一个用于扩展模板引擎的能力的系统。

（以上引用自维基百科）

# 2 准备工作
## 2.1. 安装

我学习Django的时候都是看这本[The Django Book](http://code.google.com/p/luaforwindows/downloads/list)，很适合快速入门，强烈推荐。说明：以下都是基于Windows7，[Python 2.7.6](http://www.python.org/download/releases/2.7.6/)和[Django 1.6.1](https://www.djangoproject.com/download/)，安装的时候注意一下系统和框架的版本。Django请按照官网所给的方式安装，数据库则以MySQL为例，其他如有出入则参照上面的Django Book吧。

安装完成后打开python解释器，输入以下语句，若显示版本号则安装成功。

    >>> import django
    >>> django.VERSION

## 2.2 创建Django项目

Django安装成功后，会自带一个django-admin.py的管理脚本（默认已添加到系统变量PATH中），如果控制台无法直接运行，可以到`python安装目录\Scripts`中找到这个文件。运行命令`django-admin.py startproject mysite`。这样会在你的当前目录下创建一个目录mysite。

startproject 命令创建的目录，包含4个文件：__init__.py（让Python把该目录当成一个开发包 ，即一组模块所需的文件。 这是一个空文件，一般你不需要修改它）、manage.py（一种命令行工具，允许你以多种方式与该 Django 项目进行交互）、settings.py（该Django项目的设置或配置）、urls.py（Django项目的URL设置，可视其为你的django网站的目录），

创建完成之后，在mysite目录下运行`python manage.py runserver`启动一个本地服务器，然后在浏览器中访问`http://127.0.0.1:8000/`，就可以看到新创建项目的初始页面了。

接下来一章将讲解Django最基本的URL配置。

# 3 URL配置

首先我们在项目文件夹中创建views.py文件，以备后用。

    # views.py    
    from django.http import HttpResponse

    def hello_view(request):
        return HttpResponse("Hello world")

我们创建了一个最简单最基本的视图，只有一个名为hello的函数，并返回一个HttpResponse对象。尽管我们实际在项目中不会这么写，但它足以支持Django基本的运行：在页面显示一句“Hello world”。接下来的URL配置才是重点。

URL配置就像是Django所支撑网站的目录。它的本质是URL模式以及要为该URL模式调用的视图函数之间的映射表。你就是以这种方式告诉Django，对于这个URL调用这段代码，对于那个URL调用那段代码。例如，当用户访问/foo/时，调用视图函数foo_view()，这个视图函数存在于Python模块文件view.py中。编辑urls.py并输入以下内容。

    from django.conf.urls.defaults import *
    from mysite.views import hello_view

    urlpatterns = patterns('',
        ('^hello/$', hello_view),
    )

只要在浏览器中访问`http://127.0.0.1:8000/hello`就可以看到你的第一个页面了。这里面最关键的就是用正则表达式来匹配URL，上面的模式包含了一个尖号(^)和一个美元符号((<span>$</span>)，分别表示对字符串的头部和尾部进行匹配。例如“^hello/”，那么任何以/hello/开头的如“/hello/foo”的URL将会匹配。又比如，“^$”代表一个空串，匹配根URL。

大多数动态web应用程序，URL通常都包含有相关的参数，例如你点击某个帖子跳转到该帖子的详细页面（/list/123），这里可以使用d+来匹配1个以上的数字，用\d{1,2}来匹配一或两位数字，对应的配置如下：

    urlpatterns = patterns('',
        ('^$', index_view),
        ('^hello/$', hello_view),
        ('^list/\d{1,2}/$', list_view)
    )

当这样设置URL的时候，相应的view函数应该增加一个参数来接收，如上例所示的view函数形如`def list_view(request, id): requestId = int(id)`。

这就是URL的基本配置，结合其他正则表达式的表达方法基本可以满足日常所需，更详细的请参见[这里](http://djangobook.py3k.cn/2.0/chapter03/)
