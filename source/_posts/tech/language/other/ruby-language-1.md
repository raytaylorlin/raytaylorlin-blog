title: Ruby语言特性（上）
date: 2015-01-05 21:21:34
categories:
- 技术
- 编程语言
- 其它语言
tags:
- Ruby
- 编程范型
---
Ruby是一种解释型、面向对象、动态类型的语言。Ruby采取的策略是在灵活性和运行时安全之间寻找平衡点。随着Rails框架的出现，Ruby也在2006年前后一鸣惊人，同时也指引人们重新找回编程乐趣。尽管从执行速度上说，Ruby谈不上有多高效，但它却能让程序员的编程效率大幅提高。本文将讲述Ruby语言的基础语言特性，包括基本的语法及代码块和类的定义。

<!-- more -->

# 1. 基础

在Ruby交互命令行中输入以下命令（`>>`为命令行提示符，`=>`为返回值；下文将把`=>`符号和语句写在一行内表明其返回值）：

    >> puts 'hello, world'
    hello, world
    => nil

    >> language = 'Ruby'
    => "Ruby"

    >> puts "hello, #{language}"
    hello, Ruby
    => nil

以上代码使用`puts`输出，给变量赋值，并用`#{}`的语法实现字符串替换。这表明Ruby是解释执行的；变量无需声明即可直接初始化和赋值；每条Ruby代码都会返回某个值；单引号包含的字符串表示它将直接被解释，双引号包含的字符串会引发字符串替换。

## 1.1 编程模型

Ruby是一门纯面向对象语言，在Ruby中一切皆为对象，可以用“.”调用对象具有的方法，可以通过`class`和`methods`方法查看对象的类型及支持的方法，如`4.class => Fixnum`，`7.methods => ["inspect", "%", "<<", "numerator", ...]`，`false.class => FalseClass`（方括号表示数组）。

## 1.2 流程控制

条件判断有正常的块形式，也有简单明了的单行形式；除了常见的if语句外，还有unless语句（等价于if not，但可读性更强）。同理，循环也有正常的块形式和单行形式。注意：**除了nil和false之外，其他值都代表true，包括0！**

    # 块形式
    if x == 4
      puts 'This is 4.'
    end
    # 单行形式
    puts 'This is false.' unless true
    x = x + 1 while x < 10 # x的结果为10
    x = x - 1 until x == 1 # x的结果为1

和其他C家族的语言差不多，Ruby的逻辑运算符`and`（`&&`）、`or`（`||`）都自带短路功能，若想执行整个表达式，可以用`&`或`|`

## 1.3 鸭子类型

执行`4 + 'four'`会出现TypeError的错误，说明Ruby是强类型语言，在发生类型冲突时，将得到一个错误。如果把个语句放在`def...end`函数定义中，则只有在调用函数时才会报错，说明Ruby在运行时而非编译时进行类型检查，这称为**动态类型**。Ruby的类型系统有自己的潜在优势，即多个类不必继承自相同的父类就能以“多态”的方式使用：

    a = ['100', 100.0]
    puts a[0].to_i  # => 100
    puts a[1].to_i  # => 100

这就是所谓的“鸭子类型”（duck typing）。数组的第一个元素是String类型，第二个元素是Float类型，但转换成整数用的都是`to_i`。鸭子类型并不在乎其内在类型是什么，只要一个对象像鸭子一样走路，像鸭子一样嘎嘎叫，那它就是只鸭子。在面向对象设计思想中，有一个重要原则：对接口编码，不对实现编码。如果利用鸭子类型，实现这一原则只需极少的额外工作，就能轻松完成。

## 1.4 函数

    def tell_the_truth
      true
    end

每个函数都会返回结果，如果没有显式指定返回值，函数就将退出函数前最后处理的表达式的值返回。函数也是个对象，可以作为参数传给其他函数。

## 1.5 数组

和Python一样，Ruby的数组也是用中括号来定义，如`animals = ['lion', 'tiger', 'bear']`；负数下标可以返回倒数的元素，如`animals[-1] => "bear"`；通过指定一个Range对象来获取一个区段的元素，如`animals[1..2] => ['tiger', 'bear']`。此外，数组元素可以互不相同，多为数组也不过是数组的数组。数组拥有极其丰富的API，可用其实现队列、链表、栈、集合等等。

## 1.6 散列表

    numbers = {2 => 'two', 5 => 'five'}
    stuff = {:array => [1, 2, 3], :string => 'Hi, mom!'}
    # stuff[:string] => "Hi, mom!"

散列表可以带任何类型的键，上述代码的stuff的键较为特殊——它是一个符号（symbol），前面带有冒号标识符。符号在给事物和概念命名时很好用，例如两个同值字符串在物理上不同，但相同的符号却是同一物理对象，可以通过反复调用`'i am string'.object_id`和`:symbol.object_id`来观察。另外，当散列表用作函数最后一个参数时，大括号可有可无，如`tell_the_truth :profession => :lawyer`。

# 2. 面向对象

## 2.1 代码块

代码块是没有名字的函数（匿名函数），可以用作参数传递给函数。代码块只占一行时用大括号包起来，占多行是用do/end包起来，可以带若干个参数。

    3.times {puts 'hehe'}  # 输出3行hehe
    ['lion', 'tiger', 'bear'].each {|animal| puts animal} # 输出列表的内容

上面的`times`实际上是Fixnum类型的方法，要自己实现这样一个方法非常容易：

    class Fixnum
      def my_times
        i = self
          while i > 0
            i = i - 1
            yield
        end
      end
    end
    3.my_times {puts 'hehe'}  # 输出3行hehe

这段代码打开一个现有的类，向其中添加一个自定义的`my_times`方法，并用`yield`调用代码块。在Ruby中，代码块不仅可用于循环，还可用于延迟执行，即代码块中的行为只有等到调用相关的yield时才会执行。代码块充斥于Ruby的各种库，小到文件的每一行，大到在集合上进行各种复杂操作，都是由代码块来完成的。

## 2.2 类

调用一个对象的`class`方法可以查看其类型，调用`superclass`可以查看这个类型的父类。下图展示了数字的继承链，其中横向箭头表示右边是左边实例化的对象，纵向箭头表示下边继承于上边。Ruby的一切事物都有一个共同的祖先Object。

![Ruby数字的继承链](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/Script/Ruby数字的继承链.png)

最后通过一个完整的实例——定义一棵树，来看下Ruby的类如何定义和使用，该注意的点都写在注释里面了。

    class Tree
      # 定义实例变量，使用attr或attr_accessor关键字，前者定义变量和访问变量的同名getter方法（即只读），后者定义的变量多了同名setter方法（注意这里使用了符号）
      attr_accessor :children, :node_name

      # 构造方法（构造方法必须命名为initialize）
      def initialize(name, children=[])
        @node_name = name
        @children = children
      end

      # 遍历所有节点并执行代码块block，注意参数前加一个&表示将代码块作为闭包传递给函数
      def visit_all(&block)
        visit &block
        children.each {|c| c.visit_all &block}
      end

      # 访问一个节点并执行代码块block
      def visit(&block)
        block.call self
      end
    end

    ruby_tree = Tree.new("Ruby", 
      [Tree.new("Reia"),
       Tree.new("MacRuby")])
    # 访问一个节点
    ruby_tree.visit {|node| puts node.node_name}
    # 访问整棵树
    ruby_tree.visit_all {|node| puts "Node: #{node.node_name}"}

最后再提一下Ruby的命名规范：

* 类采用CamelCase命名法
* 实例变量（一个对象有一个值）前必须加上@，类变量（一个类有一个值）前必须加上@@
* 变量和方法名全小写用下划线命名法，如underscore_style
* 常量采用全大写下划线命名法，如ALL_CAPS_STYLE
* 用于逻辑测试的函数和方法一般要加上问号，如if test?


参考资料：《七周七语言》第2章Ruby