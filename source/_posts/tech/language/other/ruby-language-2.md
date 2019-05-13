title: Ruby语言特性（下）
date: 2015-01-09 16:22:50
categories:
- 技术
- 编程语言
- 其它语言
tags:
- Ruby
- 编程范型
---
本文将接着上一篇[《Ruby语言特性（上）》](/tech/language/other/ruby-language-1/)讲述Ruby语言的核心语言特性，包括Mixin、模块、开放类等等，并使用Ruby来定义自己的语法。这些特性都是重点和难点，透过这些特性可以感受到Ruby是一门灵活性极高的语言。

<!-- more -->

# 1. 模块与混入（Mixin）

面向对象语言利用继承，将行为传播到相似的对象上。若一个对象像继承多种行为，一种做法是用多继承，如C++；Java采用接口解决这一问题，Ruby采用模块Mixin。模块是函数和常量的集合，若在类中包含一个模块，那么该模块的行为和常量也会成为类的一部分。
  
    # 定义模块ToFile
    module ToFile
      # 获取文件名
      def filename
        "object_name.txt"
      end

      # 创建文件
      def to_f
        File.open(filename, 'w') {|f| f.write(to_s)}  # 注意这里to_s在其他地方定义！
      end
    end

    # 定义用户类
    class Person
      include ToFile
      attr_accessor :name

      def initialize(name)
        @name = name
      end

      def to_s
        name
      end
    end

    Person.new('matz').to_f  # 创建了一个文件object_name.txt，里面包含内容matz

上面的代码很好理解，只是有一点要注意：`to_s`在模块中使用，在类中实现，但定义模块的时候，实现它的类甚至还没有定义。这正是鸭子类型的精髓所在。**写入文件的能力，和Person这个类没有一点关系（一个类就应该做属于它自己的事情）**，但实际开发又需要把Person类写入文件这种额外功能，这时候mixin就可以轻松胜任这种要求。

Ruby有两个重要的mixin：枚举（enumerable）和比较（comparable）。若想让类可枚举，必须实现each方法；若想让类可比较，必须实现`<=>`（太空船）操作符（比较a,b两操作数，返回1、0或-1）。Ruby的字符串可以这样比较：`'begin' <=> 'end => -1`。数组有很多好用的方法：

    a = [5, 3, 4, 1]
    a.sort => [1, 3, 4, 5]  # 整数已通过Fixnum类实现太空船操作符，因此可比较可排序
    a.any? {|i| i > 4} => true
    a.all? {|i| i > 0} => true
    a.collect {|i| i * 2} => [10, 6, 8, 2]
    a.select {|i| i % 2 == 0} => [4]
    a.member?(2) => false
    a.inject {|product, i| product * i} => 60  # 第一个参数是代码块上一次执行的结果，若不设初始值，则使用列表第一个值作为初始值

# 2. 元编程（metaprogramming）

所谓元编程，说白了就是“写能写程序的程序”，这说起来有点拗口，下面会通过实例来讲解。

## 2.1 开放类

可以重定义Ruby中的任何类，并给它们扩充任何你想要的方法，甚至能让Ruby完全瘫痪，比如重定义Class.new方法。对于开发类来说，这种权衡主要考虑了自由，有这种重定义任何类或对象的自由，就能写出即为通俗易懂的代码，但也要明白，自由越大、能力越强，担负的责任也越重。

    class Numeric
      def inches
        self
      end
      def feet
        self * 12.inches
      end
      def miles
        self * 5280.feet
      end
      def back
        self * -1
      end
      def forward
        self
      end
    end

上面的代码通过开放Numeric类，就可以像这样采用最简单的语法实现用英寸表示距离：`puts 10.miles.back`，`puts 2.feet.forward`。

## 2.2 使用method_missing

Ruby找不到某个方法时，会调用一个特殊的回调方法`method_missing`显示诊断信息。通过覆盖这个特殊方法，可以实现一些非常有趣且强大的功能。下面这个示例展示了如何用简洁的语法来实现罗马数字。

    class Roman
      # 覆盖self.method_missing方法
      def self.method_missing name, *args
        roman = name.to_s
        roman.gsub!("IV", "IIII")
        roman.gsub!("IX", "VIIII")
        roman.gsub!("XL", "XXXX")
        roman.gsub!("XC", "LXXXX")

        (roman.count("I") +
         roman.count("V") * 5 +
         roman.count("X") * 10 +
         roman.count("L") * 50 +
         roman.count("C") * 100)
      end
    end

    puts Roman.III  # => 3
    puts Roman.XII  # => 12

我们没有给Roman类定义什么实际的方法，但已经可以Roman类来表示任何罗马数字！其原理就是在没有找到定义方法时，把方法名称和参数传给`method_missing`执行。首先调用`to_s`把方法名转为字符串，然后将罗马数字“左减”特殊形式转换为“右加”形式（更容易计数），最后统计各个符号的个数和加权。

当然，如此强有力的工具也有其代价：类调试起来会更加困难，因为Ruby再也不会告诉你找不到某个方法。因此`method_missing`是一把双刃剑，它确实可以让语法大大简化，但是要以人为地加强程序的健壮性为前提。

## 2.3 使用模块

Ruby最流行的元编程方式，非模块莫属。下面的代码讲述如何用模块的方式扩展一个可以读取csv文件的类。

    module ActsAsCsv

      # 只要某个模块被另一模块include，就会调用被include模块的included方法
      def self.included(base)
        base.extend ClassMethods
      end

      module ClassMethods
        def acts_as_csv
          include InstanceMethods
        end
      end

      module InstanceMethods
        attr_accessor :headers, :csv_contents

        def initialize
          read
        end

        def read
          @csv_contents = []
          filename = self.class.to_s.downcase + '.txt'
          file = File.new(filename)
          @headers = file.gets.chomp.split(', ')  # String的chomp方法去除字符串末尾的回车换行符
          file.each do |row|
            @csv_contents << row.chomp.split(', ')
          end
        end
      end

    end  # end of module ActsAsCsv

    class RubyCsv    # 没有继承，可以自由添加
      include ActsAsCsv
      acts_as_csv
    end

    m = RubyCsv.new
    puts m.headers.inspect
    puts m.csv_contents.inspect

上述代码中RubyCsv包含了ActsAsCsv，所以ActsAsCsv的included方法中，base就指RubyCsv，ActsAsCsv模块给RubyCsv类添加了唯一一个类方法`acts_as_csv`，这个方法又打开RubyCsv类，并在类中包含了所有实例方法。如此这般，就写了一个会写程序的程序（通过模块来动态添加类方法）。

一些出色的Ruby框架，如Builder和ActiveRecord，都会为了改善可读性而特别依赖元编程。借助元编程的威力，可以做到尽量缩短正确的Ruby语法与日常用于之间的距离。注意一切都是为了提升代码可读性而服务。

# 3. 总结

Ruby的纯面向对象可以让你用一致的方式来处理对象。鸭子类型根据对象可提供的方法，而不是对象的继承层次，实现了更切合实际的多态设计。Ruby的模块和开放类，使程序员能把行为紧密结合到语法上，大大超越了类中定义的传统方法和实例变量。

核心优势：

* **优雅的语法和强大的灵活性**
* 脚本：Ruby是一门梦幻般的脚本语言，可以出色地完成许多任务。Ruby许多语法糖可以大幅提高生产效率，各种各样的库和gem（Ruby包）可以满足绝大多数日常需要。
* Web开发：很多人学Ruby最终就是为了用Ruby on Rails框架来进行Web开发。作为一个极其成功的MVC框架，其有着广泛的社区支持及优雅的语法。Twitter最初就是用Ruby实现的，借助Ruby无比强大的生产力，可以快速地开发出一个可推向市场的合格产品。

不足之处：

* **性能**：这是Ruby的最大弱点。随着时代的发展，Ruby的速度确实是越来越快。当然，Ruby是创建目的为了改善程序员的体验，在对性能要求不高的应用场景下，性能换来生产效率的大幅提升无疑是值得的。
* 并发和面向对象编程：面向对象是建立在状态包装一系列行为的基础上，但通常状态是会改变的。程序中存在并发时，这种编程策略就会引发严重问题。
* 类型安全：静态类型可提供一整套工具，可以更轻松地构造语法树，也因此能实现各种IDE。对Ruby这种动态类型语言来说，实现IDE就困难得多。

参考资料：《七周七语言》第2章Ruby