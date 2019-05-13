title: Django学习笔记（4）——Django的Admin后台管理
date: 2014-2-16 18:07:55
categories:
- 技术
- 编程语言
- Python
tags:
- Django
- Python
---
对于某一类网站，管理界面是基础设施中非常重要的一部分。这是以网页和有限的可信任管理者为基础的界面，它可以让你添加，编辑和删除网站内容。如果你用过Wordpress这款博客管理工具，一定对其管理后台很熟悉——用这个界面发布，管理博客和目录等等。但是管理界面有一问题：创建它太繁琐。当你开发对公众的功能时，网页开发是有趣的，但是创建管理界面通常是千篇一律的——你必须认证用户，显示并管理表格，验证输入的有效性诸如此类，这些都是很繁琐而且很重复的劳动。

Django在对这些繁琐和重复的工作进行了相当大的改进，它用不能再少的代码为你做了所有的一切。本文将介绍Django的Admin自动管理界面——它读取你模式中的元数据，然后提供给你一个强大而且可以使用的界面，网站管理者可以用它立即工作。

<!-- more -->

# 1. 想清楚用不用管理界面

在开始使用Django提供的这个强大的工具之前，有必要先认清Django Admin能干什么，应该什么时候用它什么时候不用它。

Django的管理界面对非技术用户要输入他们的数据时特别有用，在Django最开始开发的新闻报道的行业应用中，有一个典型的在线自来水的水质专题报道应用，它的实现流程是这样的：
1. 负责这个报道的记者和要处理数据的开发者碰头，提供一些数据给开发者
2. 开发者围绕这些数据设计模型然后配置一个管理界面给记者
3. 记者检查管理界面，尽早指出缺少或多余的字段。开发者来回地修改模块
4. 当模块认可后，记者就开始用管理界面输入数据。同时，程序员可以专注于开发公众访问视图和模板

也就是说，Django的管理界面为内容输入人员和编程人员都提供了便利的工具。此外，管理界面在下面这些情景中也是很有用的：

* 检查模块：当你定义好了若干个模块，在管理页面中把他们调出来然后输入一些虚假的数据，这是相当有用的。有时候，它能显示数据建模的错误或者模块中其它问题。
* 管理既得数据：如果你的应用程序依赖外部数据（来自用户输入或网络爬虫），管理界面提供了一个便捷的途径，让你检查和编辑那些数据。你可以把它看作是一个功能不那么强大，但是很方便的数据库命令行工具。
* 临时的数据管理程序：你可以用管理工具建立自己的轻量级数据管理程序，比如说开销记录。如果你正在根据自己的，而不是公众的需要开发些什么，那么管理界面可以带给你很大的帮助。从这个意义上讲，你可以把它看作是一个增强的关系型电子表格。

# 2. 激活管理界面

Django管理站点完全是可选择的，因为仅仅某些特殊类型的站点才需要这些功能。这意味着你需要在你的项目中花费几个步骤去激活它。

首先将`django.contrib.admin`加入setting.py的INSTALLED_APPS配置中；保证INSTALLED_APPS中包含`django.contrib.auth`，`django.contrib.contenttypes`和`django.contrib.sessions`；MIDDLEWARE_CLASSES包含`django.middleware.common.CommonMiddleware` 、`django.contrib.sessions.middleware.SessionMiddleware` 和`django.contrib.auth.middleware.AuthenticationMiddleware` 。

然后，运行`python manage.py syncdb`，这一步将生成管理界面使用的额外数据库表。 当你把“django.contrib.auth”加进INSTALLED_APPS后，第一次运行syncdb命令时, 系统会请你创建一个超级用户。如果你不这么做，你需要运行`python manage.py createsuperuser`来另外创建一个admin的用户帐号，否则你将不能登入admin。

最后，修改urls.py将admin访问配置在URLconf中，如下所示。

    from django.contrib import admin
    admin.autodiscover()
    urlpatterns = patterns('',
        (r'^admin/', include(admin.site.urls)),
    )

当这一切都配置好后，启动开发服务器(`python manage.py runserver`)，然后在浏览器中访问http://localhost:8000/admin/，现在你将发现Django管理工具可以运行了。

# 3. 使用Admin管理你的数据

使用刚刚创建的超级账户登录admin后，你会发现仅有两个默认的管理编辑模块：用户组(Groups)和用户(Users)。因为你是用超级用户登录的，你可以创建，编辑和删除任何对象。然而，不同的环境要求有不同的权限，系统不允许所有人都是超级用户。管理工具有一个用户权限系统，通过设置用户组和用户，你可以根据用户的需要来指定他们的权限，从而达到部分访问系统的目的。

现在将定义的Models加入到Admin管理中，就可以直接通过这个界面来管理数据库。这里我们引用[学习笔记（2）](/Tech/Script/Python/django-note-2/)中定义的模型Publisher、Author和Book。在books这个app的目录下，创建`admin.py`文件，然后输入以下代码：

    from django.contrib import admin
    from mysite.books.models import Publisher, Author, Book

    admin.site.register(Publisher)
    admin.site.register(Author)
    admin.site.register(Book)

这些代码通知管理工具为这些模块逐一提供界面。完成后打开admin页面，你将可以看到一个Books区域，其中包含Authors、Books和Publishers等管理区，然后你就可以对数据进行增删查改了，所见即所得。同时admin管理界面也很智能地处理了各种外键关系，例如Book模型的publisher字段后面有一个绿色加号，可以打开一个新窗口让你快速添加相关记录，然后再回头编辑；另外，如果你的字段有类型限制（比如EmailField，max_length等），在保存修改的时候会检查管理员的输入是否符合格式，方便吧！

最后再介绍一下自定义列表，这个功能可以很方便地修改admin界面特定模型的管理方式。例如在Book模型中，有title、author、publisher、publication_date四个字段，默认的admin管理会把所有字段都显示出来。看看以下代码：

    class BookAdmin(admin.ModelAdmin):
        # 指定要显示的字段
        list_display = ('title', 'publisher', 'publication_date')
        # 指定列表过滤器，右边将会出现一个快捷的日期过滤选项，以方便开发人员快速地定位到想要的数据，同样你也可以指定非日期型类型的字段
        list_filter = ('publication_date',)
        # 指定要搜索的字段，将会出现一个搜索框让管理员搜索关键词
        search_fields = ('title',)

Admin的自定义列表和自定义编辑表单还是挺强大的，详情可以参照[这里](http://djangobook.py3k.cn/2.0/chapter06/)，此处不再赘述。有了Admin这个工具，对于一些产品展示或者内容展示类型的网站，Web开发者再也不用费力气去开发一个庞大的后台管理，而可以把更多的精力投入到业务逻辑和前端开发中。