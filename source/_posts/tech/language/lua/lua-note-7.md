title: Lua学习笔记（7）——面向对象编程
date: 2015-05-11 11:10:08
categories:
- 技术
- 编程语言
- Lua
tags:
- Lua
---
Lua中的table就是一种对象，因为它和对象一样可以拥有状态，也拥有一个独立于其值的标识（一个self），也和对象一样具有独立于创建者的生命周期。但是Lua中没有类的概念，只能用元表来实现原型，用原型来模拟类和继承等面向对象特性。本文将介绍Lua关于面向对象编程的内容。

<!-- more -->

# 1. 对象与类

## 1.1 self与冒号语法

使用self参数是所有面向对象语言的一个核心，Lua只需使用冒号语法，就能隐藏该参数，例如下面两段代码是等价的。

    Account = {balance=0}
    funtion Account.withdraw(self, v)
        self.balance = self.balance - v
    end
    a1 = Account; Account = nil
    a1.withdraw(a1, 100.0)  -- 注意这是可以运行的

    function Account:withdraw(v)
        self.balance = self.balance - v
    end
    a2 = Account
    a2:withdraw(100.0)  -- 省略了a2参数传入

## 1.2 类的编写

在一些基于原型的语言中，对象是没有类型的，但每个对象都有一个原型。原型是一种常规的对象，当其他对象遇到一个未知操作时，原型会先查找它。在这种语言中要表示一个类，只需创建一个专用做其他对象的原型。Lua中实现原型很简单，只需用元表的`__index`来实现继承。

（当访问一个table中不存在的字段key时，一般得到结果为nil。事实上，访问会促使解释器去查找一个叫`__index`的元方法，如果没有这个元方法，则访问结果如前述的nil，否则由这个元方法来提供结果。元方法除了是一个函数，还可以是一个table，如果是table则直接返回该table中key对应的内容。）

如果有两个对象a和b，要让b作为a的一个原型，只需`setmetatable(a, {__index=b})`。a就会在b中查找它没有的操作。

    function Account:new(o)
        o = o or {}  -- 如果用户没有提供table，则创建一个
        setmetatable(o, self)
        self.__index = self
        return o
    end

当调用`a = Account:new{balance = 0}`时，a会将Account（函数中的self）作为其元表。当调用`a:withdraw(100.0)`时，Lua无法在table a中找到条目withdraw，则进一步搜索元表的`__index`条目，即`getmetatable(a).__index.withdraw(a, 100.0)`。由于new方法中做了`self.__index = self`，所以上面的表达式又等价于`Account.withdraw(a, 100.0)`，这样就传入了a作为self参数，又调用了Account类的withdraw函数。**这种创建对象的方式不仅可以作用于方法，还可以作用于所有其他新对象中没有的字段。**

## 1.3 继承

现在要从Account类派生出一个子类SpecialAccount（以使客户能够透支），只需：

    SpecialAccount = Account:new()
    s = SpecialAccount:new{limit=1000.00}

SpecialAccount从Account继承了new，当执行`SpecialAccount:new`时，其self参数为SpecialAccount，因此s的元表为SpecialAccount。当调用s不存在的字段时，会向上查找，也可以编写新的重名方法覆盖父类方法。

## 1.4 多重继承

上面介绍中为`__index`元方法赋值一个table实现了单继承，如果要实现多重继承，可以让`__index`字段成为一个函数，在该函数中搜索多个基类的方法字段。由于这种搜索具有一定复杂性，多重继承的性能不如单一继承。还有一种改进性能的简单做法是将继承的方法复制到子类中，但这种做法的缺点是当系统运行后就较难修改方法的定义，因为这些修改不会沿着继承体系向下传播。

## 1.5 私密性

Lua在设计对象时，没有提供私密性机制（private），但其各种元机制使得程序员可以模拟对象的访问控制。这种实现不常用，因此只做基本的了解：通过两个table来表示一个对象，一个用来保存对象的状态，一个用于对象的操作（即接口）。

    function newAccount(initialBalance)
        local self = {balance = initialBalance}
        local withdraw = function(v)
            self.balance = self.balance -v
        end
        return {
            withdraw = withdraw
        }
    end

通过闭包的方式，将具有私密性的字段（如balance）保存在self table中，并只公开了withdraw接口，这样就能实现私密性机制。

参考文献：电子工业出版社《Lua程序设计（第2版）》第16章