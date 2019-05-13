title: Django学习笔记（2）——Django的模型
date: 2014-2-3 15:21:11
categories:
- 技术
- 编程语言
- Python
tags:
- Django
- Python
---
在Django中，视图负责处理一些业务逻辑，然后返回响应结果。在当代Web应用中，业务逻辑经常牵涉到与数据库的交互，在后台连接数据库服务器，从中取出一些数据，然后在Web页面用漂亮的格式展示这些数据。这个网站也可能会向访问者提供修改数据库数据的方法。

在这一篇博文中，我们将以MySQL数据库为例，先看看不使用Django模型的数据库查询方法，然后开始学习Django的模型。

<!-- more -->

# 1. 不使用模型的数据库查询方法

假如我们不采用Django的模型，如何从数据库中获取数据呢？通常的做法就是在视图（View）层中连接数据库，写SQL语句查询数据库，然后把数据渲染给浏览器。在本例的视图中，我们使用了MySQLdb类库（可以从[这里](http://www.djangoproject.com/r/python-mysql/)获得）来连接MySQL数据库，取回一些记录，将它们提供给模板以显示一个网页代码看起来像下面这样：

    # view.py
    from django.http import HttpResponse
    import MySQLdb

    def book_list_view(request):
        db = MySQLdb.connect(user='me', db='mydb', passwd='secret', host='localhost')
        cursor = db.cursor()
        cursor.execute('SELECT name FROM books ORDER BY name')
        names = [row[0] for row in cursor.fetchall()]
        db.close()
        return HttpResponse(names)

这个方法可以达成目标，但存在着一些问题：将数据库连接参数硬编码于代码之中。 理想情况下，这些参数应当保存在 Django 配置中；我们不得不重复同样的代码：创建数据库连接、创建数据库游标、执行某个语句、然后关闭数据库；它把我们栓死在MySQL之上。如果过段时间，我们要从MySQL换到PostgreSQL，就不得不使用不同的数据库适配器（例如psycopg而不是MySQLdb），改变连接参数，根据SQL语句的类型可能还要修改SQL。理想情况下，应对所使用的数据库服务器进行抽象，这样一来只在一处修改即可变换数据库服务器。

接下来我们将开始学习Django中的模型，它提供了一种优雅的方法来抽象数据库层面的操作。

# 2. Django的模型
## 2.1 数据库配置

假定你已经完成了数据库服务器的安装，并且已经在其中创建了数据库（我是直接装了WAMP，然后启动MySQL服务）。那么只需要在settings.py文件里面设置连接数据库的参数即可，下面是以MySQL为例。

    DATABASE_ENGINE = 'mysql'
    DATABASE_NAME = 'db_my_database'
    DATABASE_USER = 'root'
    DATABASE_PASSWORD = '123456'
    DATABASE_HOST = '127.0.0.1'
    DATABASE_PORT = '3306'

输入下面这些命令来测试你的数据库配置：

    from django.db import connection
    cursor = connection.cursor()

如果没有显示什么错误信息，那么你的数据库配置是正确的。否则，你就得查看错误信息来纠正错误。详情可参考[这里](http://djangobook.py3k.cn/2.0/chapter05/)的表5-2。

## 2.2 创建一个app

Django规定，如果要使用模型，必须要创建一个app。在上一篇中我们创建的是project，project和app的区别就是：一个project包含很多个Django app以及对它们的配置，一个app是一套Django功能的集合，通常包括模型和视图，按Python的包结构的方式存在。下面的语句创建了一个名为books的app，这个目录包含了这个app的模型models.py和视图views.py，它们都是空的。

    python manage.py startapp books

## 2.3 定义模型

下面我们围绕一个关于书的例子来设计数据库。想象一下这样一个最简化的保存书籍信息的数据库：*书*有“书名”“作者”“出版社”“出版日期”，*作者*有“名字”“年龄”“Email”，*出版社*有“名字”“地址”，其中*书*可以有多个*作者*（多对多关系），但只能有一个*出版社*（一对多关系）。

如果你之前接触过数据库，对这些一定不陌生。首先你应该会设计3个表分别表示*书*、*作者*、*出版社*，然后开始着手写创建表的SQL语句（如果你要像我一样偷懒，甚至直接开个Navicat就开始创建表了），接下来就是常见的增删查改，你在你的网站的逻辑中写各种“INSERT INTO”“DELETE”“SELECT”“UPDATE”SQL语句来操作数据库。当网站的规模增长到一定程度，你可能会觉得不断重复写各种SQL语句会显得特别繁琐。下面我们来看看Django是如何定义模型的。

    # models.py
    from django.db import models

    class Publisher(models.Model):
        name = models.CharField(max_length=30)
        address = models.CharField(max_length=50, blank = True)  # blank为True表明可空

    class Author(models.Model):
        name = models.CharField(max_length=30)
        age = models.IntegerField()
        email = models.EmailField()

    class Book(models.Model):
        title = models.CharField(max_length=100)
        author = models.ManyToManyField(Author)
        publisher = models.ForeignKey(Publisher)
        publication_date = models.DateField()

看着这段代码，你可以这样理解：这里定义的每个类都继承了models.Model，代表数据库中的表；类里面的字段代表数据表中的字段，数据类型则由CharField（相当于varchar）、DateField（相当于datetime）这样显而易见的类加上属性（如max_length限定长度）来定义；表与表之间的关系则由ManyToManyField（多对多）、ForeignKey（外键，一对多）和OneToOneFiled（一对一）来限定。

接下来在settings.py中找到INSTALLED_APPS这一项，如下：

    INSTALLED_APPS = (
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.sites',
        'mysite.books',   # 加上这一句，mysite是project，books是app
    )

在命令行中运行`python manage.py syncdb`，看到几行“Creating table...”的字样，你的数据表就创建好了。只要你定义了模型的类和字段，Django将全部自动帮你创建好数据库。

syncdb命令是同步你的模型到数据库的一个简单方法。它会根据INSTALLED_APPS里设置的app来检查数据库，如果表不存在，它就会创建它。需要注意的是，syncdb并不能将模型的修改或删除同步到数据库；如果你修改或删除了一个模型，并想把它提交到数据库，syncdb并不会做出任何处理。如果你觉得这很不方便，可以使用一个Django的第三方的app——South——来同步数据库，使用方法可参考[这篇文章](http://www.cnblogs.com/BeginMan/p/3324774.html)

# 3. 常用的数据库操作

下面直接在代码里面注释说明，Django设计的简洁可以让你一看代码就知道怎么通过模型去访问数据库。

    from books.models import Publisher

    # 先创建对象，然后再save，相当于SQL中的INSERT
    p1 = Publisher(name='Apress', address='2855 Telegraph Avenue')
    p1.save()
    # 注意：尽管我们没有在models给表设置主键，但是Django会自动添加一个id作为主键

    # 通过objects这个模型管理器的all()获得所有数据行，相当于SQL中的SELECT * FROM
    publisher_list = Publisher.objects.all()
    for publisher in publisher_list:
        # 对每个publisher做处理

    # 修改其中一个字段，再save，相当于SQL中的UPDATE（但这里会更新所有列）
    p1.name = 'Apress Publishing'
    p1.save()
    # 如果只需要更新一列，可以用update方法，并应用于结果集上更新多个数据
    Publisher.objects.all().update(address='USA')

    # 删除掉数据库中的一行，相当于SQL中的DELETE
    p1.delete()

    # filter相当于SQL中的WHERE，可设置条件过滤结果
    p2 = Publisher.objects.filter(name='Apress')    #单条件查询
    p2 = Publisher.objects.filter(name='Apress', address__contains='Telegraph')    # 多条件查询，双下划线表明会进行一些魔术般的操作，contains部分会被翻译成WHERE address LIKE '%Telegraph%'

    # 用get来获取单个结果，注意如果没有结果或结果不唯一会抛异常
    try:
        p = Publisher.objects.get(name='Apress')
    except Publisher.DoesNotExist:
        print "Apress isn't in the database yet."
    else:
        print "Apress is in the database."

    # 按name字段来排序，相当于SQL中的ORDER BY，如果换成"-name"则是逆序
    result = Publisher.objects.order_by("name")

    # 上面的方法可以连锁使用
    Publisher.objects.filter(country="U.S.A.").order_by("-name")

    # 用索引限定数据行数，相当于SQL中的OFFSET 0 LIMIT 2
    Publisher.objects.order_by('name')[0:2]

    # 直接通过字段外键来访问数据，这有点类似于SQL中的JOIN
    from books.models import Book
    publisher_addr = Book.objects.get(id=1).publisher.address

    # 对于用"ForeignKey"定义的关系，关系的另一端也能反向追溯回来。通过字段名加_set的方式可以访问到
    p = Publisher.objects.get(name='Apress Publishing')
    p.book_set.all()
    # 注意到Publisher类里面没有定义book这个字段，但因为定义了外键，Django已经帮我们生成了book_set这个属性

    # 多对多和外键工作方式相同，只不过我们处理的是QuerySet而不是模型实例
    b = Book.objects.get(id=1).author.all()
    # 反向查询还是字段名加_set，例如要查看一个作者的所有书籍
    a = Author.objects.get(name='Adrian')
    a.book_set.all()

# 4. 结语

从上面的介绍可以看到Django的模型已经帮我们干了非常多的事情，比如我们可以通过访问对象的属性和方法来操作数据库，这种技术叫做[对象关系映射](http://zh.wikipedia.org/wiki/%E5%AF%B9%E8%B1%A1%E5%85%B3%E7%B3%BB%E6%98%A0%E5%B0%84)（ORM，Object Relational Mapping）。本文仅仅介绍了Django模型最基础的部分，其他高级的用法比如增加额外的Manager方法，添加修改数据库字段等应用可以参照[这里](http://djangobook.py3k.cn/2.0/chapter10/)。虽然Django给我们提供了如此方便的工具，但它还是需要一定的学习成本的，比如以前你简简单单写个SQL语句就可以搞定的事情，现在要转换为对象的思维，然后去定义模型，学习模型的API。而且听别人说，在涉及到比较复杂的数据库查询和优化的时候，ORM会显得力不从心，所幸Django可以支持原始SQL查询（同样请参见上面的链接）。在日常使用和中小型应用中，使用Django的ORM已经游刃有余，毕竟开发效率才是王道。






