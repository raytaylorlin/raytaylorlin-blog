title: Io语言特性（上）
date: 2015-01-11 21:21:34
categories:
- 技术
- 编程语言
- 其它语言
tags:
- Io language
- 编程范型
---
Io同Javascript、Lua一样，是一种原型语言，这意味着每个对象都是另一个对象的复制品。如今的大多数Io社区，都致力于把这门语言作为带有微型虚拟机和丰富并发特性的可嵌入语言来推广。它的简单语法和原型编程模型都值得我们重点关注，在了解Io之后，也可以让你对Javascript的运行机制的理解变得更透彻。

<!-- more -->

# 1. 基础

## 1.1 创建对象

面向对象语言中，通常都是通过对某个类调用new创建一个新对象，但在原型语言Io中，不区分类和对象，而是通过复制现有对象创建新对象，现有对象就是原型。

    Io> "Hello, io" print    // 输出Hello, io

    Io> Vehicle := Object clone
    ==> Vehicle_0x1003b61f8:
      type = "Vehicle"
    Io> Vehicle description := "Something to take you places"
    ==> "Something to take you places"
    // Vehicle nonexistingSlot = "This won't work" 对不存在的槽使用=号赋值会报错
    Io> Vehicle description
    ==> Something to take you places
    Io> Vehicle slotNames
    ==> list("type", "description")

1. 把print消息发送给字符串就可以输出那个字符串，Io中所有的操作都是发送消息，接收者在左边，消息在右边。
2. Object是根对象，我们发送clone消息过去，它会返回一个新对象，并将其赋值给Vehicle。注意这里的Vehicle不是类，也不是创建对象的模板，而是实实在在的对象。
3. 对象还带有槽（slot），可以把一组槽想象成散列表，通过键可以引用到任何一个槽；可以用`:=`或`=`给槽赋值，当槽不存在时，前者可以创建出一个槽，后者则会抛出一个异常。
4. 通过向对象发送槽的名字，可以获取槽中的值。`slotNames`槽是内置的，可以获取一个槽名列表。

## 1.2 原型和继承

继续上面的代码：

    Io> Car := Vehicle clone
    Io> Car slotNames
    ==> list("type")
    Io> Car description
    ==> "Something to take you places"  // Car没有description槽，所以Io把description消息转发给Car的原型Vehicle，并在Vehicle中找到这个槽
    Io> ferrari := Car clone
    Io> ferrari slotNames
    ==> list()  // 这下连type槽都没了，因为依照Io的惯例，其类型应以大写字母开头
    Io> ferrari type
    ==> Car  // 去父对象调用type槽，得到原型的类型

以大写字母开头的对象时类型，具有type槽，而类型的复制品则以小写字母开头。注意：**类型仅仅是帮助Io程序员更好地组织代码的工具，不管是大写开头还是小写开头，它们统统都是对象，Io中没有类。**

    Io> method("I'm method." println)  // 创建一个方法
    ==> method(...)
    Io> method() type  // 方法也是对象，因此可以获取其类型
    ==> Block
    Io> Car drive := method("Vroom" println)  // 方法可以赋值给一个槽
    ==> method(...)
    Io> ferrari drive  // 调用槽就会调用对应方法
    Vroom
    ==> Vroom
    Io> ferrari getSlot("drive")  // getSlot可以获取槽的内容
    Io> ferrari proto  // proto槽输出该对象的原型信息
    ==> Vehicle_0x1003b61f8: ...
    Io> Lobby  // Lobby是主命名空间，包含所有已命名对象
    ==> Object_0x1002184e0: ...

## 1.3 集合

Io有几种类型的集合：List列表对象是任意类型对象的有序集合，Map对象时键值对的原型，如同Ruby的散列表一样。

    Io> todos := list(1, 2, 3)  // 由Object对象的list方法可以把传入参数包装起来创建列表
    Io> todos size  // 获取列表大小
    Io> todos append(4)  // 追加元素

    Io> elvis := Map clone  // 映射没有语法糖，所以只能从Map clone出来
    Io> elvis atPut("home", "Graceland")  // 创建一个键值对
    Io> elvis at("home") ==> Graceland
    Io> elvis asObject  // 将散列表转换为对象，key则相应转换为对象的槽

## 1.4 true、false、nil和单例

Io条件判断和其他语言基本一致，都有`true`、`false`、`and`、`or`等关键字。注意：和Ruby一样0是true。有趣的地方在于，调用`true clone`依旧会返回`true`，`false`和`nil`也一样，这三个东西都是单例，对它们进行复制，返回的就是单例对象本身的值。要创建一个单例非常简单：

    Singleton := Object clone
    Singleton clone := Singleton

通过重定义Singleton的clone槽，让其返回自身，而不是像往常那样让请求沿着对象原型链向上传递最终到达Object对象，这样就可以实现单例。Io也是一门灵活性极高的语言，例如`Object clone := "broken"`可以使得Io再也无法创建对象，这种情况无法修复，只能终止进程。同Ruby一样，高灵活性是一把双刃剑，但如果运用得好，完全可以用几行漂亮的代码就实现一些*领域特定语言*（domain-specific language, DSL）。

# 2. 基本控制结构

## 2.1 循环和条件

    Io> loop("infinite loop", println)  // 无限循环输出，可以Ctrl+C中断
    Io> while(i <= 11, i println; i = i + 1)  // while循环接受2个参数，一个循环条件参数和一个用来求值的消息。分号可以把两个不同的消息连接起来
    Io> for(i, 1, 11, 2, i println)  // 输出1 3 5 7 9 11
    // 控制结构以函数形式实现，形式为if(condition, true code, false code)
    Io> if(true, "true", "false")
    ==> true
    Io> if(false) then("true" println) else("false" println)
    false
    ==> nil

## 2.2 运算符

在Io中，调用`OperatorTable`可以看到运算符表，可以看到赋值是另一种类型的运算符。运算符左边的数字代表优先级，参数优先绑定到优先级靠近0的运算符上。下面的代码自定义了一个运算符`xor`：

    Io> OperatorTable addOperator("xor", 11)
    ==> OperatorTable_0x100296098
    Operators
      ...
      10 && and
      11 or xor ||
      ...
    // 逻辑运算符相当于true或false的槽，用穷举法实现
    Io> true xor := method(bool, if(bool, false, true))
    Io> false xor := method(bool, if(bool, true, false))
    // 接下来就可以使用新定义的运算符了
    Io> true xor true
    ==> false
    Io> false xor true
    ==> true

最后对Io语言特性做一点小结：

* 所有事物都是**对象**
* 所有与对象交互的都是**消息**
* 你要做的不是实例化类，而是复制那些叫**原型**的对象
* 对象会记住它的原型
* 对象有**槽**
* 槽包含对象（包括方法)
* 消息返回槽中的值，或调用槽中的方法
* 若对象无法响应某消息，则把消息转发给自己的原型

参考资料：《七周七语言》第3章Io