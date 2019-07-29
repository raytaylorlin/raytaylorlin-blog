title: Cocos2d-x 3.0绘制系统
date: 2015-09-08 14:55:08
categories:
- 技术
- 游戏开发
- 客户端
tags:
- Cocos2d-x
---
在Cocos2d-x 3.0之前，Cocos2d-x每个元素的绘制逻辑都分布在每个元素内部的draw()方法里，并紧密地依赖UI树的遍历。Cocos2d-x 3.0对绘制部分进行了重构，新的架构将绘制部分从UI树的遍历中分离出来，其设计更优雅、更灵活、更易于扩展。本文将介绍Cocos2d-x 3.0新绘制系统的特点、架构及绘制细节。

<!-- more -->

# 1. 新绘制系统的特点

* 将绘制逻辑从主循环中分离。
* 采用应用程序级别的视口剪裁。如果一个UI元素在场景中的坐标位移视口之外，那么它不会发送任何绘制命令到绘制栈上。
* 采用自动批绘制技术。如果一个场景中多个不同类型的UI元素使用相同的纹理，可以只调用一次绘制命令。
* 更简单地实现绘制的自定义。

# 2. 绘制系统概览

Cocos2d-x 3.0新绘制系统分为三个阶段：生成绘制命令、对绘制命令进行排序、执行绘制命令。

首先，通过UI树的遍历给每个元素生成一个RenderCommand（定义了怎样绘制一个UI元素），并将该命令添加到renderer的绘制栈中，如下图所示。接着引擎使用`globalZOrder`及元素的遍历顺序对绘制命令进行排序。最后执行绘制命令，对一般的RenderCommand，按顺序执行，对Sprite使用的QuadCommand，若两个命令相邻且使用相同的纹理、着色器等，则会组合成一个命令（即自动批处理）。

![遍历UI树并将绘制命令发送到绘制栈](http://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/game/遍历UI树并将绘制命令发送到绘制栈.png)

## 2.1 RenderCommand概况

每个RenderCommand实例包含一个`globalOrder`属性，用于决定绘制顺序，实际上，它们几乎都来自Node的`globalZOrder`属性

5种RenderCommand类型：

* QUAD_COMMAND：根据1个纹理和4个顶点绘制一幅图片，所有绘制图片的命令都会调用到这里，处理这个类型命令的代码就是绘制贴图的OpenGL代码。
* GROUP_COMMAND：用于包装多个RenderCommand的集合，可以用来实现子元素剪裁（对应ClippingNode）和绘制子元素到纹理（对应RenderTexture）。
* BATCH_COMMAND：用于绘制一个TextureAtalas，如Label、TileMap。这种类型的命令不能参与自动批绘制。
* CUSTOM_COMMAND：自定义绘制命令
* UNKNOWN_COMMAND：未知绘制命令

## 2.2 RenderQueue和GroupCommand概况

每个UI元素的RenderCommand会被发送到一个叫RenderQueue的绘制命令栈上，Renderer持有多个RenderQueue（用`_renderGroups`来存储）。**但GroupCommand比较特殊，它只指向一个RenderQueue。可以认为一个RenderQueue就是一个GroupCommand，而创建一个GroupCommand时会将其作为一个普通的RenderCommand发送到当前的RenderQueue上，并在Renderer上创建一个新的RenderQueue。**

## 2.3 RenderCommand的排序

由于每一帧都可能执行数百个RenderCommand，所以Cocos2d-x对此进行了优化，每个RenderQueue只对其包含的**globalOrder非0**的RenderCommand进行排序，而RenderCommand被添加到RenderQueue中的顺序使由Node的`localZOrder`决定的。所以，实际上只需要对少数特殊设置了globalOrder属性的Node进行排序即可。注意，每个RenderQueue实例中实际包含了3个RenderCommand数组，分别存放globalOrder小于0、等于0和大于0的RenderCommand，这样可以最大限度地减少排序的量。

# 3. 绘制系统相关机制

## 3.1 QuadCommand

QuadCommand用于绘制一个或多个矩形区域，每个矩形是一个纹理的一部分。这是最基础的绘制命令，包含了TextureID（使用的纹理）、Shader Program、BlendFunc（混合模式）和Quads（绘制的矩形区域的定义，包括每个点的坐标、颜色和纹理坐标）4部分内容。

Cocos2d-x使用`Renderer::render()`方法进行自动批绘制的过程是：

1. 第一次遇到一个QuadCommand时不会理你绘制，而是将其放到一个数组中缓存起来，然后继续迭代
2. 若遇到第二个RenderCommand仍然是QuadCommand，并且使用相同的Material（纹理、着色器、混合模式等等），则继续添加到缓存数组，若不是，则首先绘制之前的缓存数组的指令。这样就能实现自动合并绘制命令。

如何判断是否是相同的Material？`QuadCommand::generateMaterialID()`方法检查是否包含自定义的着色器（包含自定义着色器就不能参与批绘制），如果不包含就使用与着色器名称、纹理名称及混合方程相关的参数计算一个Hash值，Hash值相同表明是相同的Material。

## 3.2 元素可见性

在OpenGL ES的图元装配阶段，渲染管线会对每个图元执行视锥体裁剪操作，位于视锥体之外的图元会被丢弃或裁剪。所谓的自动裁剪（Auto Culling）技术，是在遍历UI树时对Sprite进行位置计算，如果发现其位于屏幕之外，则不会发送绘制命令到Renderer中。Node类还有一个visible属性，用于控制一个元素是否显示，如果为false，则该元素在遍历UI树时会被忽略。

如果一个应用程序有很大的应用场景，则不应该完全依赖自动裁剪。因为自动裁剪只是减少了绘制命令调用的次数，而这些元素所使用的纹理仍然占据着内存，所以还要注意对纹理内存的管理。

## 3.3 绘制时机

将绘制和UI树遍历分离带来一个问题：我们不知道元素什么时候被绘制了，我们只有等到下一帧才能确定所有绘制命令被执行了。这种机制对一些操作（如RenderTexture需要等到绘制完毕后操作纹理）显得很不方便，一般有两种方法来处理这种情况：

1. 注册一个一次性Schedule，在下一帧被执行时读取上一帧的绘制结果，并注销该Schedule。
2. 若要精确把握绘制时机，可以添加一个CustomCommand，将其func属性重写为不包含GL命令调用的自定义回调。这样只要把CustomCommand放在合适的绘制位置（通过globalOrder或localZOrder来调节）。像RenderTexture中的saveToFile方法就是采用这种方法来控制绘制时机。

参考文献：电子工业出版社《我所理解的Cocos2d-x》第4章