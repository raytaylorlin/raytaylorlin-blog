title: 新版Hexo主题Raytaylorism v2发布
date: 2016-03-18 11:28:32
categories:
- 日常
tags:
- Hexo
- 博客
---
时隔两年多，我自制的Hexo主题Raytaylorism（[Github地址](https://github.com/raytaylorlin/hexo-theme-raytaylorism)）终于喜迎v2版本的发布。这个项目可以说是我在学生时代Web前端领域中的收官之作，因此在本次升级主题的过程中，一切都尽量做到精细。例如所有的页面都经过重新设计，采用清新的的响应式的Material Design风格，加入了个性化的“读书”“关于”页面，以及皮肤自定义、分类目录、正文滚动目录、打赏等等特色功能，并且该主题支持最新的Hexo 3.1版本。由于主题的功能较为复杂，所有的安装说明和配置事项都写在了Github项目的README中，需要使用主题的同学请**认真仔细阅读README**哦，**特别是[启用](https://github.com/raytaylorlin/hexo-theme-raytaylorism#启用重要)那一节的说明很重要！很重要！很重要！一定要照做否则你会发现hexo启动不起来或最终效果和截图上的不一样。** 使用过程中有任何问题欢迎给我开issue。下面正文将介绍主题在Github上没有详细解释但又非常有特色的功能。

<!-- more -->

# 1. 换肤功能

我个人非常喜欢Material Design这种简洁清新的设计风格，也非常喜欢其定义的各种颜色。博客主题是一种非常个性化的东西，我喜欢的配色方案（包括主题默认的indigo-pink方案）不一定是你喜欢的，因此raytaylorism在几乎所有带颜色的区域都预留了配置的接口（具体参见[README-样式-主题颜色配置](https://github.com/raytaylorlin/hexo-theme-raytaylorism#样式)）。不过，总有一些设计感不强的同学不知道如何下手，所以下面给出了3款参考的皮肤配置方案，大家可以各取所需随意发挥。配色的一个基本原则，就是选好一种主色和强调色。

## 1.1 夏日甜橙

主色：blue 强调色：deep-orange

![夏日甜橙](http://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image%2Fdaily%2Fblue_deeporange.jpg)

    color:
        header: indigo
        footer: indigo
        page_nav: indigo
        side_nav: indigo darken-1
        tag: green accent-4
        link: indigo
        pagination: green
        tab: green
        archive_item: grey
        fab: green
        fab_2: blue
        fab_3: orange
        new: pink
        about_header: indigo
        about_title: indigo

## 1.2 绿野仙踪

主色：green 强调色：red

![绿野仙踪](http://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image%2Fdaily%2Fgreen_red.jpg)

    color:
        header: green
        footer: green
        page_nav: green
        side_nav: green darken-1
        tag: red lighten-1
        article_title_link: green
        link: red
        pagination: red
        tab: red
        archive_item: grey
        fab: red
        fab_2: cyan
        fab_3: light-green
        new: red
        about_header: green
        about_title: green

## 1.3 林地木屋

主色：brown 强调色：light-green

![林地木屋](http://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image%2Fdaily%2Fbrown_lightgreen.jpg)

    color:
        header: brown darken-1
        footer: brown darken-1
        page_nav: brown darken-1
        side_nav: brown darken-1
        tag: light-green
        article_title_link: brown
        link: light-green
        pagination: light-green
        tab: light-green
        archive_item: grey
        fab: light-green
        fab_2: red
        fab_3: purple
        new: light-green
        about_header: brown darken-1
        about_title: brown

# 2. 文章分类目录

如果点击我的博客菜单中的“分类”按钮，会发现左侧侧滑栏会出来一个带有多个层级的文章分类列表。

![主题的分类目录](http://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image%2Fdaily%2Fraytaylorism-category.png)

如果你也想要得到类似的效果（包括标签tag），需要按照[hexo官方的categories/tags说明文档](https://hexo.io/docs/front-matter.html#Categories-amp-Tags)给你的博客文章设置正确的`categories`和`tags`配置项。也就是说你的每一篇博文的markdown文件中，需要设置类似于下方的几行配置：

    categories:
    - 一级目录
    - 二级目录
    - 三级目录
    tags:
    - 第一个标签
    - 第二个标签

准确设置后，启动hexo时主题会自动解析所有文章并形成分类树，最终生成上图那样的分类层次。

# 3. 读书页面

![读书页面截图](http://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image%2Fdaily%2Fraytaylorism-reading.png)

在[README-数据-读书页面](https://github.com/raytaylorlin/hexo-theme-raytaylorism#数据)中其实已经说明了如何配置读书页面的数据，照着例子来改就行了。值得注意的是，“已读”“在读”“想读”这些标签的文字是可以通过`reading.json`中的`define`字段来修改的，甚至你还可以仿照读书页面，扩展出自己的专属页面，例如“作品”页面等。只要是满足这种列表条目的数据均可以在其上自由发挥。

# 4. 关于页面

![关于页面截图](http://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image%2Fdaily%2Fraytaylorism-about.png)

在[README-数据-关于页面](https://github.com/raytaylorlin/hexo-theme-raytaylorism#数据)中也说明了个`about.json`中各个字段的含义，照着例子改就行了。

另外关于页面的末尾还有一个“打赏”功能，点开后会出现微信和支付宝的二维码，这个需要你自己去制作自己的付款二维码，然后把`reward`字段的两个图片链接替换掉。如果暂时不需要的话请将该字段设为null，**千万不要傻乎乎地把主题照搬让别人给我的账号打赏了=。=**

关于主题任何使用上的疑问，欢迎在[Issues](https://github.com/raytaylorlin/hexo-theme-raytaylorism/issues)上提问。
