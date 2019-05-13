title: Node.js学习笔记：模块机制
date: 2014-10-08 16:22:53
categories:
- 技术
- Web前端
- NodeJS
tags:
- Node.js
- 模块化
---

Javascript自诞生以来，曾经没有人拿它当做一门编程语言。在Web 1.0时代，这种脚本语言主要被用来做表单验证和网页特效。直到Web 2.0时代，前端工程师利用它大大提升了网页上的用户体验，JS才被广泛重视起来。在JS逐渐流行的过程中，它大致经历了工具类库、组件库、前端框架、前端应用的变迁。Javascript先天就缺乏一项功能：模块，而CommonJS规范的出现则弥补了这一缺陷。本文将介绍CommonJS规范及Node的模块机制。

<!-- more -->

在其他高级语言中，Java有类文件，Python有import机制，PHP有include和require。而JS通过&lt;script&gt;标签引入代码的方式显得杂乱无章。过去人们不得不用命名空间等方式来人为地约束代码，直到[CommonJS](http://wiki.commonjs.org/wiki/CommonJS)规范的出现，前后端的Javascript才得以实现大一统。Node借鉴了CommonJS的Modules规范实现了一套非常易用的模块系统。

# 1. CommonJS模块规范

CommonJS的模块规范分为3个部分：

1. 模块引用：通过`require()`方法并传入一个**模块标识**来引入一个模块的API到当前上下文中，如`var math = require('math');`
2. 模块定义：通过`exports`对象来导出当前模块的方法或变量。模块中还存在一个`module`对象，exports实际上是module的属性。在Node中，一个文件就是一个模块，模块内的“全局变量”对外都不可见，只有挂载在exports上的属性才是公开的，如`exports.add = function() {}; exports.PI = 3.1415926;`
3. 模块标识：实际上就是传递给`require()`的参数，如上述的`'math'`，它必须是符合camel命名法的字符串，或者是以“.”“..”开头的相对路径或绝对路径，它可以没有文件名后缀“.js”

# 2. Node模块实现过程

在Node中，模块分为两类：一类是Node本身提供的核心模块，另一类是用户自己编写的文件模块。核心模块有一部分在Node源代码的编译过程中，编译成了二进制文件，在Node启动时核心模块就被直接加载进内存中，所以它的加载速度是最快的。文件模块则是在运行时动态加载，需要经历三个步骤：路径分析，文件定位，编译执行。**注意，Node对引入过的模块都会进行缓存，以减少二次引入时的开销，并对相同模块的二次加载都采用最优先从缓存加载的策略。**

## 2.1 路径分析

路径分析主要分析上述提到的模块标识符，主要分为以下几类：

* 核心模块，如http、fs、path等
* .或..开始的相对路径文件模块
* 以/开始的绝对路径文件模块
* 自定义文件模块，可能是一个文件或包的形式。Node会根据*模块路径*数组`module.paths`来逐个尝试查找目标文件，通常是沿着当前目录逐级向上直到根目录查找名为`node_modules`的目录，所以这是查找最费时的一种方式。

## 2.2 文件定位

在路径分析的基础上，文件定位需要注意如下细节：

* 文件扩展名分析：由于CommonJS规范允许模块标识不填写扩展名，Node会按.js、.json、.node的次序不足扩展名，依次尝试
* 目录分析和包：若通过上述文件扩展名分析后没有查找到对应文件，却得到一个目录，Node会把目录当做一个包来处理

## 2.3 编译执行

定位到具体文件后，Node会新建一个模块对象，根据路径载入并编译。对于不同的扩展名，载入方法有所不同：

* .js文件：通过fs模块同步读取文件并编译执行
* .node文件：这是用C/C++编写的扩展文件，通过`dlopen()`方法加载
* .json文件：通过fs模块同步读取文件，用`JSON.parse()`解析返回结果
* 其余扩展名文件：都被当做.js文件载入

我们知道每个模块文件中默认都存在着require、exports、module这3个变量，甚至在Node的API文档中，我们知道每个模块还有__filename、__dirname这2个变量的存在，它们是从何而来的呢？Node的模块又是怎么做到声明的“全局变量”实际上是不会污染到其他模块的？事实上，**Node在编译JS模块过程中会对文件内容进行头尾包装**。下面是一个JS文件经过头尾包装的例子：

    (function(exports, require, module, __filename, __dirname) {
        /* 中间是JS文件的实际内容 */
        var math = require('math');
        exports.area = function(radius) {
            return Math.PI * radius * radius;
        };
        /* JS文件的实际内容结束 */
    });

这样每个模块文件之间都进行了作用域隔离，同时require、exports、module等变量也被注入到了模块的上下文当中。这就是Node对CommonJS模块规范的实现。关于C/C++模块及Node核心模块的编译过程较为复杂，不再赘述。

# 3. 模块调用栈

有必要明确一下Node中各种模块的调用关系，如下图所示：

![Node模块间调用关系](http://raytaylorlin-blog.qiniudn.com/image%2Fnodejs%2FNode%E6%A8%A1%E5%9D%97%E9%97%B4%E8%B0%83%E7%94%A8%E5%85%B3%E7%B3%BB.jpg)

C/C++内建模块是最底层的模块，属于核心模块，主要提供API给Javascript核心模块和第三方Javascript文件模块调用，实际中几乎不会接触到此类模块。Javascript核心模块主要职责有两种：一种是作为C/C++内建模块的封装层和桥接层供文件模块调用，另一种是纯粹的功能模块，不需要跟底层打交道。文件模块通常由第三方编写，包括普通Javascript模块和C/C++扩展模块。

# 4. 包与NPM

## 4.1 包结构

包本质上是一个存档文件（一般为.zip或.tar.gz），安装后解压还原为目录。CommonJS的包规范由包结构和包描述文件两部分组成。一个完全符合CommonJS规范的包结构应包含以下文件：

* package.json：包描述文件
* bin：存放可执行二进制文件的目录
* lib：存放Javascript代码的目录
* doc：存放文档的目录
* test：存放单元测试用例的目录

## 4.2 包描述文件

包描述文件是一个JSON文件——package.json，位于包的根目录下，是包的重要组成部分，用于描述包的概况信息。后面要提到的NPM的所有行为都与这个文件的字段息息相关。下面将以知名Web框架express项目的[package.json](https://github.com/strongloop/express/blob/master/package.json)文件为例说明一些常用字段的含义。

* name：包名
* description：包简介
* version：版本号，需遵照“语义化的版本控制”，参照[http://semver.org/](http://semver.org/)
* dependencies：使用当前包所需要依赖的包列表。这个属性十分重要，NPM会通过这个属性自动加载依赖的包
* repositories：托管源代码的位置列表

其余字段的用法可以参照[NPM package.json说明](https://www.npmjs.org/doc/files/package.json.html)

## 4.3 NPM常用功能

NPM（node package manager），通常称为node包管理器。它的主要功能就是管理node包，包括：安装、卸载、更新、查看、搜索、发布等。

### 4.3.1 NPM包安装

Node包的安装分两种：本地安装、全局安装。两者的区别如下：

* 本地安装`npm install <package-name>`：package会被下载到当前所在目录，也只能在当前目录下使用。
* 全局安装`npm install -g <package-name>`：package会被下载到到特定的系统目录下，安装的package能够在所有目录下使用。

### 4.3.2 NPM包管理

下面以grunt-cli（grunt命令行工具）为例，列出常用的包管理命令：

* `npm install`：安装package.json文件的dependencies和devDependencies字段声明的所有包
* `npm install grunt-cli@0.1.9`：安装特定版本的grunt-cli
* `npm install grunt-contrib-copy --save`：安装grunt-contrib-copy，同时保存该依赖到package.json文件
* `npm uninstall grunt-cli`：卸载包
* `npm list`：查看安装了哪些包
* `npm publish <folder>`：发布包

参考资料：《深入浅出NodeJS》第二章