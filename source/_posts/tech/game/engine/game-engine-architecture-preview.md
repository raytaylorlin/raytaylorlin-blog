title: 游戏引擎架构总览
date: 2016-05-27 17:07:53
categories:
- 技术
- 游戏开发
- 游戏引擎
tags:
- 游戏引擎
- 架构
---
游戏引擎通常由运行时组件和工具套件两部分构成。本文先探讨运行时部分的架构，给出了一个不包含工具的极其庞大的总览图（如果时间有限仅看此图即可），并对图中每一组件进行描述，最后再阐述工具方面的内容。如同所有软件系统，游戏引擎也是以软件层构建的，而且通常上层依赖下层，下层不依赖上层。

<!-- more -->

# 1. 运行时引擎架构

下图为游戏运行时引擎架构的总览图，本图相对原书标上了序号，以方便下面各节阐述时对应其位置，并省略了一些细节的组成部分。**图片较大，建议在新标签页中打开图片查看。**

![游戏运行时引擎架构](http://raytaylorlin-blog.qiniudn.com/image%2Fengine%2F%E6%B8%B8%E6%88%8F%E8%BF%90%E8%A1%8C%E6%97%B6%E5%BC%95%E6%93%8E%E6%9E%B6%E6%9E%84.png)

## 1.1 硬件与操作系统

图中的A1为目标硬件层，代表用来执行游戏的计算机系统或游戏主机。典型平台包括基于微软Windows或Linux的PC、苹果的iPhone及Machintosh、微软的Xbox360、索尼的PS4/PSP、任天堂的NDS/Wii等等。

图中的A2是设备驱动程序，是由操作系统或硬件厂商提供的最低阶软件组件。驱动程序负责管理硬件资源，也隔离了操作系统及上层引擎，使上层的软件无须理解不同硬件版本的通信细节差异。

图中的A3是操作系统。在PC上操作系统是一直运行的，PC游戏不能假设拥有硬件的所有控制权；而在游戏主机上，操作系统通常只是个轻量级的库，链接到游戏的执行档里。不过像Xbox 360和PS3这些新主机中，操作系统也会中断游戏的执行，接管某些系统资源以显示在线信息。

## 1.2 第三方软件开发包和中间件

图中的B表明大多数游戏引擎会借用第三方SDK及中间件（middleware），SDK提供基于函数或基于类的API。下面是一些常见的例子：

* 数据结构及算法：如STL、STLport、Boost、Loki。因为PC上有虚拟内存系统，所以可以无碍使用STL，而游戏主机上，只有有限的（甚至没有）虚拟内存功能，而且缓存命中失败的代价极高，所以最好编写自定义的数据结构
* 图形：如DirextX、OpenGL、libgcm、Edge等
* 碰撞和物理：如Havok、PhysX、ODE（Open Dynamics Engine）等
* 角色动画：如Granny、Havok Animation、Edge等
* 人工智能：如Kynapse，提供低阶的AI构件，例如路径搜寻、静态和动态物体回避、空间内的脆弱点辨认，以及相当好的AI和动画间接口
* 生物力学角色模型：如Endorphin、Euphoria等，利用了真实人类运动的高阶生物力学模型，去产生角色动作

## 1.3 平台独立层

图中的C为平台独立层。大部分游戏引擎需要运行于不同的平台上，该层包装了常用的标准C语言库、操作系统调用及其他基础API，确保包装了的接口在所有硬件平台上均为一致。

## 1.4 核心系统

游戏引擎以及其他大规模复杂C++应用软件，都需要一些有用的实用软件（utility)，统称为“核心系统”，即图中的D。以下是一些核心系统层的常见功能：

* 内存管理：几乎每个游戏引擎都有若干个自定义内存分配系统，以保证高速的内存分配及释放，并控制内存碎片所造成的负面影响
* 数学库：游戏本质上就是高度数学密集的，所以每个游戏引擎都有若干个数学库，提供矢量、矩阵、四元数旋转、三角学、数值积分、解方程组，以及其他游戏程序员需要的功能
* 自定义数据结构及算法：除非引擎设计者想完全依靠第三方软件包，否则引擎通常要提供一组工具去管理基础数据结构和算法，以减少或完全消去动态内存分配，并保证在目标平台上的运行效率为最优

## 1.5 资源管理

图中的E为资源管理器，提供一组统一的接口去访问任何类型的游戏资产及其他引擎输入数据。有些引擎使用高度集中及一致的方式（例如虚幻的包package、OGRE的ResourceManager类）。其他引擎使用专案（ad hoc）方法，比如让程序员直接读取磁盘的或压缩的文件（如雷神之锤引擎使用的PAK文件）。

## 1.6 渲染引擎

任何游戏引擎中，渲染引擎是最大及最复杂的组件之一。渲染器有很多不同的架构方式，通常采用分层架构。

* 低阶渲染器（图中F1）：包含引擎中全部原始的渲染功能，着重于高速渲染丰富的几何图元集合
    * 图形设备接口：使用图形SDK（如DirectX及OpenGL），都需要编写不少代码去枚举图形设备，初始化设备，建立渲染表面等，这些工作通常由图形设备接口组件负责
    * 其他渲染器组件：目的是要收集须提交的几何图元，包括网格、线表、点表、例子、地形块、字符串等等。低阶渲染器还提供视区（viewport）抽象、材质系统及动态光照系统
* 场景图/剔除优化（图中F2）：该层基于某些可视性判别算法去限制低阶渲染器提交的图元数量。非常小的游戏世界可能只需要简单的平截头体剔除算法，比较大的游戏世界则可能需要较高阶的空间细分数据结构，令渲染更有效率
* 视觉效果（图中F3）：支持广泛的视觉效果，例如粒子系统（烟、火、水花等）、贴花系统（弹孔、脚印等），还有一些全屏幕后期处理，例如高动态范围光照（HDR）、敷霜效果、全屏抗锯齿（FSAA）、颜色校正等等
* 前端（图中F4）：该层主要用于显示2D图形，如平视显示器（HUD）、GUI界面等等，通常会用附有纹理的四边形结合正射投影来渲染，或者用完全三维的四边形公告板（billboard）渲染

## 1.7 剖析与调试工具

图中的G用于剖析调优性能，分析内存。还包含了游戏内置调试功能，包括调试用绘图、内置菜单、主控台、录制回放游戏过程等。市场上有很多优良的通用软件剖析工具，如VTune、Quantify、Purify等等，但是多数游戏也会加入自制的剖析与调试工具以应对特殊需求。

## 1.8 碰撞和物理

图中的H为碰撞与物理组件。游戏中如果没有碰撞检测，物体会互相穿透，并且无法在虚拟世界里合理地互动。碰撞和物理系统一般是紧密联系的，因为当碰撞发生时，碰撞几乎总是由物理积分及约束满足逻辑来解决。一些游戏还包含真实或半真实的刚体动力学模拟。时至今日，游戏引擎通常使用第三方的物理SDK，如Havok、PhysX和ODE。

## 1.9 动画

图中的I为动画系统，游戏常会用到精灵/纹理动画、刚体层次结构动画、骨骼动画、每顶点动画、变形目标动画5种基本动画。现今游戏
中，骨骼动画是最盛行的动画方式。此外，骨骼网格渲染组件是连接渲染器和动画系统的桥梁，这些组件合作渲染的过程称为蒙皮（skinning）。

当使用布娃娃系统时，动画和物理系统便产生紧密耦合，这是因为布娃娃是无力的（经常是死了的）角色，其运动完全由物理系统模拟。物理系统把布娃娃当作受约束的刚体系统，用模拟来决定身体每部分的位置及方向。

## 1.10 人体学接口设备

图中的J用于处理玩家输入，包括键盘鼠标、游戏手柄及其他专用游戏控制器（如方向盘、跳舞毯、Wii遥控器等）。除了输入功能，一些设备也提供输出，如游戏手柄的震动、Wii遥控器的音频输出等。

在架构HID引擎时，通常让硬件的低阶细节与高阶游戏操作脱钩。HID引擎从硬件取得原始数据，为控制器的每个摇杆设置环绕中心点的死
区，去除按钮抖动，检测按下和释放按钮事件，演绎加速计的输入并使该输入平滑。HID引擎也可能包含一个系统，负责检测弦（chord）（即数个按钮一起按下）、序列（即接钮在时限内顺序按下）、手势（即按钮、摇杆、加速计等输入的序列）。

## 1.11 音频

图中的K为音频引擎。一些游戏团队会为这些引擎加入自定义功能，或用内部方案替换，例如微软为DirectX平台提供一个名为XACT的优秀的音频工具包，艺电也开发了内部的音频引擎SoundR!OT。然而，即使游戏团队用既有的音频引擎，开发每个游戏时仍然需要大量的定制软件开发、整合工作及注意细节，才可以制作出有高质量音频的最终产品。

## 1.12 在线多人/网络

图中的L为在线多人/网络组件。多人游戏有单屏多人、切割平多人、网络多人、大型多人在线等多种基本形式。支持多人游戏，会深切影响到游戏世界对象模型、人体学接口设备系统、玩家控制系统、动画系统等多个组件的设计。把一个现有的担任引擎改装成多人引擎的难度是非常大的，但如果反过来则比较简单——许多游戏引擎把单人游戏模式当做是一个玩家参与的多人游戏。

## 1.13 游戏性基础系统

游戏性（gameplay）是指：游戏内进行的活动、支配游戏虚拟世界的规则、玩家角色的能力（也称为玩家机制）、其他角色和对象的能力、玩家的长短期目标。游戏性编程除了用引擎的原生语言，通常还会使用高阶的脚本语言，为了连接低阶的引擎子系统和游戏性代码，多数游戏引擎会引入一个软件层，即图中的M。

* 游戏世界和游戏对象模型：游戏世界含动态与静态的元素，而典型的游戏对象有静态背景几何物体（如建筑、地形）、动态刚体（如石头、椅子）、玩家角色、NPC、武器、抛射物、载具、光源、摄像机
* 事件系统：事件驱动架构常用于游戏对象间的通信
* 脚本系统：使用脚本语言来编写游戏独有的游戏性规则和内容，可以快速开发，避免重新编译链接
* 人工智能基础：像Kynapse这种商用AI引擎，抽象了大多数AI系统共有的模式，在这个基础层上可以很容易地开发个别游戏的逻辑。其功能包括用路径节点和漫游体积组成网络定义AI角色可行走的地区和路径，在漫游地区边界周围的简化碰撞信息，A*路径搜寻，联系碰撞系统及世界模型进行视线追踪及其他感知，AI决策层架构等等

## 1.14 个别游戏专用子系统

如图中的N，每个游戏都有若干自身特有的游戏性系统。如果可以清楚地分开引擎和游戏，这条分界线会位于特定游戏专用子系统和游戏性基础软件层之间。实际上，这条分界线永远不会是完美的。一些游戏的特定知识，总是会向下渗透到游戏基础软件层中，更有甚者会延伸至引擎核心。

# 2. 工具套件

## 2.1 数字内容创作工具

游戏本质上是多媒体应用。游戏引擎的输入数据形式广泛，例如三维网格数据、纹理位图、动画数据、音频文件等。所有源数据皆由美术或音效师等专业人员使用数字内容创作（Digital Content Creation，DCC）应用软件制作，如Maya、3ds Max、Photoshop、SoundForge等等。有些游戏引擎提供专门的设计游戏世界的编辑器，而有的团队会在现有软件像3ds Max的基础上开发插件去设计场景，甚至用简单的位图编辑器去制作地形高度图。总之，游戏团队想要及时开发高完成度的产品，工具必须**相对易用**，并且**绝对可靠**。

## 2.2 资产（asset）调节管道

DCC所生成的数据格式，很少有直接用于游戏中的，原因有两点：生成的数据格式通常比游戏所需的复杂得多，游戏引擎只需其中一小部分信息；直接读取速度过慢，而且有些格式是不公开的专有格式。因此，DCC软件制作的数据，通常要导出为容易读取的标准格式或自定义格式，有时还需要针对不同平台进行再处理，以便在游戏中使用。从DCC到游戏引擎的管道，就是所谓的资产调节管道。

## 2.3 常见的游戏资产数据

* 几何图形数据
    * 笔刷集合图形：由凸包集合定义，每个凸包则由多个平面定义。其优点是制作迅速简单，便于设计师建立粗略的原型，也可用作碰撞体积；缺点是分辨率低难以制作复杂图形，不能支持有关节的物体或运动角色
    * 三维模型/网格：由三角形和顶点组成，每个网格使用若干个材质。网格通常在三维建模软件里制作，并且需要专用的导出器来导出游戏引擎可读的格式
* 骨骼动画数据：骨骼网格是一种为关节动画而绑定到骨骼层次结构之上的特殊网格，游戏引擎需要网格本身、骨骼层次结构和若干动画片段3种数据来渲染骨骼网络
* 音频数据：由专业的音频制作工具导出，有不同格式和采样率。音频文件通常组织成音频库，以方便管理，载入及串流。
* 粒子系统数据：由视觉特效设计师使用第三方工具（如Houdini）或引擎自带的粒子效果编辑工具制作
* 游戏世界数据：不少商用游戏引擎会提供优良的世界编辑器，用于编辑游戏世界

参考文献：电子工业出版社《游戏引擎架构》第1.6、1.7节