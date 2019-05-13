title: 游戏引擎的游戏性系统简介
date: 2016-12-21 16:45:32
categories:
- 技术
- 游戏开发
- 游戏引擎
tags:
- 游戏引擎
- 游戏性系统
---
游戏引擎是复杂的多层软件系统，而游戏的本质，并非在于其使用的引擎或技术，而是其游戏性（gameplay）。游戏机制（game mechanics）一词，可以把游戏性这个概念变得更为具体。游戏机制通常定义为一些规则，这些规则主宰了游戏中多个实体之间的互动，如定义玩家的目标、成败的准则、角色的各种能力、游戏体验的整体流程等等。本文将简单介绍用于定义及管理游戏机制的引擎系统及相关工具。

<!-- more -->

# 1. 剖析游戏世界

游戏类型虽然五花八门，但大多数会有一种基本的结构模式，通常由以下部分组成：

1. 世界元素
    * 静态元素：地形、建筑等等几乎不会动或主动与游戏性交互的物体
    * 动态元素：角色、NPC、道具、粒子特效、动态光源、区域等等
    * 区分静态和动态元素，主要是用于优化性能——静态元素都可以预先计算或忽略，减少运行时游戏世界中动态元素的运算
    * 有一些游戏含有可破坏环境，算是模糊静态和动态元素分界的例子，说明元素是否静态并不是绝对的
2. 世界组块：庞大的游戏世界通常会被拆分成为独立可玩的区域，可以演化成关卡、地图、地区等等。分割关卡有几个原因，首先是内存限制；其次它也是一个控制游戏整体流程的方便机制；最后它可以作为分工的单位，方便开发团队分别构建及管理。
3. 高级游戏流程：指由玩家**目标**所组成的序列、树或图，可演化成任务、关卡、波（如塔防波次）、胜利条件或失败惩罚。在故事驱动的游戏中，流程可能也包含多个游戏内置电影、过场动画。

# 2. 游戏世界编辑器

## 2.1 典型功能

游戏性内容对应的创作工具便是游戏世界编辑器，其用于定义世界组块，并填入静态及动态元素。所有商用游戏引擎都有某种形式的世界编辑工具，大部分会提供以下列出的主要功能。

* 世界组块创建及管理：除了组块管理基本功能外，还可以连接若干静态网格，以及AI用的导航地图、可攀抓边缘信息等等静态数据。有的还提供专门的地形编辑器用于编辑地形（或解析高度场地形）和水体。
* 可视化游戏世界：可让开发者大幅提高开发效率，通常3D游戏提供顶、侧、正视图和三维透视视图4部分，2D游戏提供正射视角。有的编辑器直接整合自制的渲染引擎至工具中，有的把自身整合至第三方3D软件，有的会通过与实际的游戏引擎通信，利用游戏引擎来渲染三维视图，甚至整合至游戏引擎本身。
* 导航：提供滚动、放大缩小、聚焦某个对象旋转、摄像机飞行模式、记录历史摄像机位置并跳转等等方便开发的功能。
* 选取：在编辑器中可以选取个别或框选多个游戏对象，并对它们批量操作。使用光线投射方式选取三维对象时，编辑器可让用户循环选取与光线相交的所有对象，而不是总选取最近者。
* 图层：把对象用预设或用户自定义的图层来分组，把游戏世界中的内容有条理地组织起来。图层也是分工的重要工具，多人可以在同个世界组块上的不同图层工作而不冲突。
* 属性网格：如下图所示，可视化编辑游戏对象的属性（一般是键值对），不仅可以编辑常见的数值和字符串，还能支持下拉框、复选框、滑块、颜色选取器等控件编辑。
    * 选取多个对象后的编辑方式：此高级特性把选中的对象的共有属性混合在一起显示。在网格中编辑公共值时，会把新值更新至所有选取对象的属性中。
    * 自由格式属性：通常这种属性集会关联到某个用户自定义的对象，以形成新的“自由格式”属性，如光源属性集包含位置、方向、颜色、强度及光源类型属性。

![属性网格示例](http://raytaylorlin-blog.qiniudn.com/image/engine/%E5%B1%9E%E6%80%A7%E7%BD%91%E6%A0%BC%E7%A4%BA%E4%BE%8B.jpg)

* 安放对象及对齐辅助工具：除了基本的平移、旋转、缩放工具外，有的编辑器还提供对齐至网格，对齐至地形，对齐至对象，多个对象分布或对齐等功能。
* 特殊对象类型
    * 光源：通常用特殊图标表示，因为光源没有网格。编辑器可能会尝试模拟光源对场景的照明效果，让设计师能实时调整并能看到场景的最终大致效果。
    * 粒子发射器：若编辑器是独立于渲染引擎之上，则可简单用图标表示，或尝试在编辑器中模拟效果；若编辑器是内置于游戏引擎，则可以实际模拟调整，达到“所见即所得”的效果。
    * 区域：即空间中的体积，供游戏侦测相关事件用（如Unity中的trigger）
* 读/写世界组块：有的引擎把每个组块储存为单个文件，有的可以独立读/写个别的图层；有的引擎使用自定义的二进制文件格式，有的使用如XML的文本格式。
* 快速迭代：优秀的编辑器会支持某程度的动态微调功能以供快速迭代。有的编辑器在游戏本身内执行，让用户即时看到改动的效果，有的连接至运行中的游戏，或完全脱机运行。具体的机制并不重要，最重要的是给用户足够短的往返迭代时间。

## 2.2 集成资产管理工具

有些引擎的编辑器会整合游戏资产数据库的其他方面功能，例如设定网格/材质的属性、设定动画/混合树/动画状态机、设置对象的碰撞/物理属性、管理材质资源等，著名的例子有UnrealEd和Unity。它们能对用户提供统一、实时、所见即所得的资产管理视图，促进快速高效的游戏开发过程。

不同的工具对资产的优化时间点也不一样。

* UnrealEd在导入资产时就会对资产优化，这样在关卡设计上能缩短法代时间，但是改动网格、动画、音频等来源资产会变得更痛苦。
* Source及雷神之锤引擎，把资产优化延后至烘焙关卡、执行游戏之前。
* 《光环（Halo）》给用户选择在任意时刻转换原始资源——这些资源在第一次载入至引擎前转换至优化格式并缓存，避免每次执行游戏时都要再做无意义的转换。

# 3. 游戏性基础系统的组件

如果可以合理地画出游戏与游戏引擎的分界线，那么游戏性基础系统就是刚刚位于该线之下。理论上，我们可以建立一个游戏性基础系统，其大部分是各个游戏皆通用的。实际上不同引擎之间有许多共有模式，以下列出一些常用组件，后续的文章就会逐渐记录这些组件的功能和设计方法。

* 运行时游戏对象（runtime game object）模型
* 实时更新对象模型
* 关卡管理及串流
* 目标及游戏流程管理
* 消息及事件处理
* 脚本

参考文献：电子工业出版社《游戏引擎架构》第13章