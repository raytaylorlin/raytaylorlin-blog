title: Io语言特性（下）
date: 2015-01-27 16:25:34
categories:
- 技术
- 编程语言
- 其它语言
tags:
- Io language
- 编程范型
---
本文将接着上一篇[《Io语言特性（上）》](/tech/language/other/io-language-1/)，讲述Io语言的消息，反射和并发。

<!-- more -->

# 1. 消息

## 1.1 消息反射与call方法

Io语言中除了注释符和参数之间的逗号外，**一切事物都是消息**。消息反射是Io一项很重要的能力，可以通过反射查询任何消息的任何特性，再对它们执行适当的操作。

消息由三部分组成：发送者（sender）、目标（target）和参数（arguments），消息由发送者发送至目标，然后由目标执行该消息。可以用`call`方法访问任何消息的元信息。

    Io> postOffice := Object clone
    // 下面的各个方法可以获取消息的各种元信息
    Io> postOffice packageSender := method(call sender)
    Io> postOffice messageTarget := method(call target)
    Io> postOffice messageArgs := method(call message arguments)
    Io> postOffice messageName := method(call message name)

    // 定义一个可以发送消息的mailer对象
    Io> mailer := Object clone
    ==> Object_0x1005bfda0    // 注意这里尾号为0bfda0是mailer对象
    Io> mailer deliver := method(postOffice packageSender)
    Io> mailer deliver
    ==> Object_0x1005bfda0:
      deliver = method(...)    // 可以看出消息发送者是deliver方法

    Io> postOffice messageTarget
    ==> Object_0x1004ce658:
      messageTarget = method(...)
      packageSender = method(...)   // 从槽名可以看出消息目标是postOffice
    Io> postOffice messageArgs("one", 2, :three)
    ==> list("one", 2, :three)    // 直接输出了参数
    Io> postOffice messageName
    ==> messageName    // 消息本身的名字

大多数语言都将参数作为栈上的值传递，例如Java在调用方法时先计算参数的每个值，然后把值放到栈上。Io就不是这样，Io传递的事消息本身和上下文，再由接收者对消息求值。Io的if，形式是`if(booleanExpression, trueBlock, falseBlock)`，现在如果要再实现一个`unless`，实现方法可以是下面这样：

    // 这种方法是不行的，因为调用unless的时候，else和then都会被马上执行，实际我们需要的是当cond为true时，执行else，否则执行then
    Object unless := method(cond, then, else,
        if(cond, else, then))
    // 正确的延迟执行实现
    Object unless := method(
        (call sender doMessage(call message argAt(0))) ifFalse(
        call sender doMessage(call message argAt(1))) ifTrue(
        call sender doMessage(call message argAt(2)))
    }
    unless(1 == 2, write("One is not two\n"), write("one is two\n"))  // 输出One is not two

上面代码中的`doMessage`可以执行任意消息，有点类似于其他语言里面的`eval`。Io会对消息参数进行解释，但会延迟绑定和执行。

## 1.2 对象反射

下面的代码给Object定义了一个ancestors方法，这个方法会沿着原型链向上查找，输出每个原型的名称以及带有的槽名，直到Object为止。

    Object ancestors := method(
        prototype := self proto
        if(prototype != Object,
            writeln("Slots of ", prototype type, "\n------------")
            prototype slotNames foreach(slotName, writeln(slotName))
            writeln
            prototype ancestors
        )
    )

1.1的例子是消息反射的例子，处理消息的发送者、目标和消息体；这个例子是对象反射的例子，处理对象和对象的槽。

# 2. 其他特性

## 2.1 领域特定语言（DSL）

Io在定义DSL方面的能力非常强大，据说用Io实现C语言的一个子集仅需约40行代码！Io提供了非常多的控制方法来解析语言本身。假如我们想要解析一个普通的`test.json`文件：

    {
        "name": "Ray Taylor",
        "lucky_numbers": [33, 6],
        "job": {
            "title": "Web Front End Developer"
        }
    }

如果是其他语言的话，可能会写一个语法分析器识别上面这段文本中不同元素，然后生成一个Io可理解的结构。但通过Io的强大特性，我们可以通过下面这段代码对Io做些改动（注释中已包含完整的解释），改动完之后Io就会认为上面这段文本的语法是正确的，并且构建出相应的散列表来。

    // 把一个“:”运算符添加到Io的赋值运算符表中，现在只要Io代码遇到“:”，就会把它转换成atPutNumber。所以遇到key:value时就会转换成atPutNumber("key", value)
    OperatorTable addAssignOperator(":", "atPutNumber")  
  
    // Io代码遇到大括号（{}），就会调用curlyBrackets方法
    curlyBrackets := method(
        // 创建一个空散列表，供存放数据
        data := Map clone
        // call message正是json大括号中的代码，arguments则是由逗号“,”分隔的参数列表。循环遍历参数列表就相当于处理json对象中的每一行
        call message arguments foreach(arg,
            // 以第1个arg（"name": "Ray Taylor"）举例，data doMessage(arg)相当于执行data "name": "Ray Taylor"，冒号翻译成atPutNumber，所以代码就相当于data atPutNumber("name", "Ray Taylor")
            data doMessage(arg)
        )
        // 最后相当于把data返回
        data
    )

    // 解析中括号（[]），原理跟上面的基本一样，不再赘述
    squareBrackets := method(
        arr := list()
        call message arguments foreach(arg,
            arr push(call sender doMessage(arg))
        )
        arr
    )  

    // 在Map对象上扩展一个atPutNumber
    Map atPutNumber := method(
        // 其实算法的核心就是调用Map atPut槽
        self atPut(
            // 注意key:value总是会转换为atPutNumber("key", value)（key有字符串包围），所以要去掉原key的头尾字符串。由于消息是不可变的，为了去掉引号，要使用asMutable转化为一个可变值
            call evalArgAt(0) asMutable removePrefix("\"") removeSuffix("\""),
            call evalArgAt(1)
        )
    )  

    // File是Io与文件交互的原型，with指定了文件名并返回一个文件对象，openForReading打开该文件并返回该文件对象，而contents返回该文件的内容
    s := File with("test.json") openForReading contents
    // doString把字符串求值为Io代码
    json := doString(s)
    json at("name") println  // Ray Taylor
    json at("lucky_numbers") println  // list(6, 13)
    json at("job") at("title") println   // Web Front End Developer

从上面的例子可以看到Io中可以随心所欲把运算符重定义为组成DSL的符号，从而改变Io的语法。

## 2.2 forward方法

**当你把消息发送给对象时，对象会计算所有参数（参数其实就是消息），获取消息的名称、目标和发送者，然后尝试用目标上的消息名称读取槽。如果槽存在，则返回其数据或触发其包含的方法；否则把消息转发给原型。**当槽名不存在时，实际会调用系统的forward方法把消息转发给原型，这有点类似Ruby的method_missing，但因为Io没有类，所以改变forward会改变从Object获得基本行为的方式。所以覆盖forward方法的风险要更高一些，但如果用得恰当，会产生非常巧妙的效果。

下面的代码将构造一种新语法来对XML进行处理。这种新语法及对应的XML如图所示：

![利用Io代码来表示XML](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/Script/利用Io代码来表示XML.png)

    Builder := Object clone
    // 覆盖forward，使其可以接收任意方法
    Builder forward := method(
        // 使用消息反射，输出开标签
        writeln("<", call message name, ">")
        // 遍历消息的每个参数
        call message arguments foreach(arg,
            // 递归调用
            content := self doMessage(arg);
            // 如果消息是个字符串（字符串的类型是Sequence序列），则直接输出
            if(content type == "Sequence", writeln(content))
        )
        // 使用消息反射，输出闭标签
        writeln("</", call message name, ">")
    )

    Builder ul(
              li("Io"),
              li("Lua"),
              li("Javascript"))   // 输出看起来像HTML ul和li标签的内容

虽然这种新语法未必比传统的XML有多大程度的提高，但这例子还是很有指导意义。你可以完全改变一个Io原型的继承运行机制，甚至定义自己的Object原型，并以这个新对象为基础创建其他原型，从而创建出一门和Io行为截然不同的新语言。

## 2.3 并发

### 2.3.1 协程

协程是并发的基础，它提供了进程的自动挂起和恢复执行的机制。可以把协程想象为带有多个入口和出口的函数，每次遇到`yield`都会自动挂起当前进程，把控制权转到另一进程中。通过在消息前加上@或@@，可以异步触发消息，前者返回future（下文讲述），后者返回nil，并在其自身线程中触发消息。

    lilei := Object clone
    lilei talk := method(
        "Hello." println
        yield
        "Fine, thank you. And you?" println
        yield
    )
    hanmeimei := Object clone
    hanmeimei talk := method(
        yield
        "How are you?" println
        yield
        "I am fine, thanks." println
    )

    // 异步触发两个人的方法
    lilei @@talk; hanmeimei @@talk
    // 这一行用来等待所有异步消息执行完毕，然后退出程序
    Coroutine currentCoroutine pause

上面这段程序不难理解。通过异步触发两个人的talk，使两个不相干的Object实例并发执行，用yield消息在指定时刻自动把控制权交给另一方法，从而让两个需要彼此协作的进程轻松实现“对话”任务。

### 2.3.2 actor

actor是通用的并发原语，它可以发送消息、处理消息以及创建其它actor。actor接收到的消息是并发的。一个actor改变其自身的状态，并通过严格控制的队列接触其它actor

参考资料：《七周七语言》第3章Io