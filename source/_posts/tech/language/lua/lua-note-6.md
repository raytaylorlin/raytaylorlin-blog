title: Lua学习笔记（6）——环境与模块
date: 2015-05-10 10:39:41
categories:
- 技术
- 编程语言
- Lua
tags:
- Lua
---
模块就是一个程序库，而包是一系列模块。Lua中可以通过require来加载模块，然后得到一个全局变量表示一个table。Lua将其所有的全局变量保存在一个被称为“环境”的常规table中。本文首先介绍环境的一些实用技术，然后介绍如何引用模块及编写模块的基本方法。

<!-- more -->

# 1. 环境

Lua将环境table保存在一个全局变量`_G`中，可以对其访问和设置。有时我们想操作一个全局变量，而它的名称却存储在另一个变量中，或者需要通过运行时的计算才能得到，可以通过`value = _G[varname]`来获得动态名字的全局变量。

关于“环境”的一大问题是它是全局的，任何对它的修改都会影响程序的所有部分。Lua 5允许每个函数拥有一个子集的环境来查找全局变量，可以通过`setfenv`来改变一个函数的环境，第一个参数若是1则表示当前函数，2则表示调用当前函数的函数（依次类推），第二个参数是一个新的环境table。

    a = 1
    setfenv(1, {})
    print(a) -- 会报错，print是一个nil。这是因为一旦改变环境，所有的全局访问都会使用新的table

为了避免上述问题，可以使用`setfenv(1, {_G = _G})`将原来的环境保存起来，然后用`_G.print`来引用。另一种组装新环境的方法是使用继承，下面的代码新环境从源环境中继承了print和a，任何赋值都发生在新的table中。

    a = 1
    local newgt = {}
    setmetatable(newgt, {__index = _G})
    setfenv(1, newgt)
    print(a)

# 2. 模块与包

## 2.1 调用模块

要调用模块mod中的foo方法，可以用`require`函数来加载，如：

    require "mod"
    mod.foo()
    -- 或者
    local m = require "mod"
    m.foo()

`require`函数的行为： （关于require使用的路径查找策略不赘述）
在`package.loaded`这个table中检查模块是否已加载 
=> 已加载，就返回相应的值（可见一个模块只会加载一次） 
=> 未加载，就试着在`package.preload`中查询传入的模块名 
===> 找到一个函数，就以该函数作为模块的加载器 
===> 找不到，则尝试从Lua文件或C程序库中加载模块 
=====> 找到Lua文件，通过`loadfile`来加载文件
=====> 找到C程序库，通过`loadlib`来加载文件

## 2.2 使用环境

下面的代码说明了如何用环境来创建一个复数（complex）模块：

    -- 模块设置
    local modname = "complex"
    local M = {}
    _G[modname] = M
    package.loaded[modname] = M

    -- 声明模块从外界所需的所有东西
    local _G = _G  -- 保留旧环境的引用，使用时需要像_G.print这样用
    local io = io

    -- 运行这句之后环境就变了
    setfenv(1, M)

    function new(r, i) return {r=r, i=i} end

    function add(c1, c2)
        return new(c1.r + c2.r, c1.i + c2.i)
    end

这样声明函数add时，就成为了`complex.add`，调用同一模块的其他函数也不需要加前缀。

## 2.3 module函数

Lua 5.1提供了一个新函数`module`，囊括了上面一系列定义环境的功能。在开始编写一个模块时，可以直接用`module("modname", package.seeall)`来取代前面的设置代码。在一个模块文件开头有这句调用后，后续所有代码都不需要限定模块名和外部名字，同样也不需要返回模块table了。

## 2.4 子模块与包

Lua支持具有层级的模块名，用一个点来分隔名称中的层级。例如一个模块名为`mod.sub`，就是mod的一个子模块。一个包（package）就是一个完整的模块树，它是Lua中发型的单位。注意，当搜索一个子模块文件时，require会把点号当做目录分隔符来搜索，也就是说调用`require "a.b"`会尝试打开`./a/b.lua`，`/usr/local/lua/a/b.lua`，`/usr/local/lua/a/b/init.lua`。通过这种加载策略，可以将包的所有模块组织到一个目录中。

参考文献：电子工业出版社《Lua程序设计（第2版）》第14-15章