title: Lua学习笔记（5）——迭代器，错误处理
date: 2015-05-07 13:05:11
categories:
- 技术
- 编程语言
- Lua
tags:
- Lua
---
本文首先介绍如何编写适用于泛型for的迭代器，再介绍Lua的编译、执行与错误处理相关的内容。

<!-- more -->

# 1. 迭代器

## 1.1 泛型for原理

迭代器是一种可以遍历集合中所有元素的机制，在Lua中通常将迭代器表示为函数，每调用一次函数，就返回集合中“下一个”元素。每个迭代器都需要在每次成功调用之间保持一些状态，这样才能知道它所在的位置及如何步进到下一个位置，closure就可以完成此项工作。下面的示例是列表的一个简单的迭代器：

    function values(t)
        local i = 0
        return function() i = i + 1; return t[i] end
    end

    -- 循环调用
    t = {10, 20, 30}
    iter = values(t)
    while true do
        local el = iter()
        if el == nil then break end
        print(el)
    end

    -- 泛型for调用
    for el in values(t) do print(el) end

泛型for为一次迭代循环做了所有的簿记工作。它在内部保存了迭代器函数，并在每次迭代时调用迭代器，在迭代器返回nil时结束循环。实际上泛型for保存了3个值：迭代器函数f、恒定状态s、控制变量a。**for做的第一件事就是对in后面的表达式求值，并返回3个值供for保存；接着for会以s和a来调用f。在循环过程中控制变量的值依次为`a1 = f(s, a0)`，`a2 = f(s, a1)`，依次类推，直至ai为nil结束循环。**

## 1.2 迭代器的状态

无状态的迭代器本身不保存任何状态，for循环只会用恒定状态和控制变量来调用迭代器函数。这类迭代器典型例子就是ipairs，下面是ipairs的Lua实现：

    local function iter(s, i)
        i = i + 1
        local v = s[i]
        if v then return i, v end
    end
    function ipairs(s)
        return iter, s, 0
    end

当for循环调用ipairs(list)时，会获得3个值，然后Lua调用iter(list, 0)得到list, list[1]，调用iter(list, 1)得到list, list[2]，知道得到一个nil为止。

虽然泛型for只提供一个恒定状态和一个控制变量用于状态的保存，但有时需要保存许多其他状态。这时可以用closure来保存，或者将所需的状态打包为一个table，并保存在恒定状态中。

# 2. 编译与错误机制

## 2.1 编译

尽管Lua是一种解释型语言，但它确实允许在运行代码前，先将代码预编译为一种中间形式。其实，**区别解释型语言的主要特征并不在于是否能编译它们，而在于编译器是否是语言运行时库的一部分，即是否有能力执行动态生成的代码。**可以说正因为存在了诸如`dofile`这样的函数，才可以将Lua称为解释型语言。

`dofile`用于运行Lua代码块，而`loadfile`会从一个文件加载Lua代码块，然后编译代码，把编译结果作为一个函数返回。要注意`loadfile`不会引发错误，它只是返回错误值但不处理错误。`dofile`的基本原理如下：

    function dofile(filename)
        local f = assert(loadfile(filename))
        return f()
    end

`loadstring`是从一个字符串读取代码，并返回一个对应的函数。注意，`loadstring`总是在全局环境中编译它的字符串。此外，这些函数不会带来任何副作用，它们只是将程序块编译为一种中间表示，然后将结果作为一个匿名函数返回。此时如果不将此匿名函数赋值给一个变量并调用，是不会产生任何结果的。

## 2.2 错误处理与异常

Lua遇到任何非预期条件都会引发一个错误，我们也可以显式地调用`error`函数并传入一个错误消息得参数来引发一个错误。像`if not <condition> then error(<anything>) end`这样的组合是非常通用的代码，所以可以用`assert(<condition>, <msg>)`来完成此类工作。另外一种处理的方式是返回错误代码（如nil）。

`pcall`函数以一种“保护模式”来调用它的第一个参数，并捕获所有执行中引发的错误。如果没有发生错误，pcall会返回true及函数调用的返回值，否则返回false及错误消息。因此可以用error来抛出一个异常或使用pcall来捕获异常，错误消息则可以标识出错误的类型或内容。

    if pcall(function()
        <受保护的代码>
    end) then
        <常规代码>
    else
        <错误处理代码>
    end

## 2.3 追溯错误

当`pcall`返回其错误消息时，它已经销毁了调用栈的部分内容（pcall到错误发生点之间的这部分调用）。而`xpcall`函数除了接受一个需要被调用的函数外，还接受一个*错误处理函数*。当发生错误时，Lua会在调用栈展开前调用这个错误处理函数，里面可以用debug库来获取错误的额外信息。如`debug.debug`会提供一个Lua提示符，让用户检查错误原因，`debug.traceback`获取当前执行的调用栈。

参考文献：电子工业出版社《Lua程序设计（第2版）》第7-8章