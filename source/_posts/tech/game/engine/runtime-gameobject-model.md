title: 运行时游戏对象模型（上）——模型架构
date: 2016-12-31 22:37:32
categories:
- 技术
- 游戏开发
- 游戏引擎
tags:
- 游戏引擎
- 游戏对象模型
---
在游戏性基础系统的各种组件中，运行时对象模型可能是最复杂的，通常它会提供这些核心功能：动态产生（spawn）和销毁（destroy）游戏对象，联系底层引擎系统，实时模拟对象行为，对象查询和引用，存档及对象持久性等等。本文将从对象模型架构开始，阐述“以对象为中心”和“以属性为中心”的两种基本架构，接着介绍几种游戏对象引用和查询的方法。

<!-- more -->

# 1. 各种运行时对象模型架构

## 1.1 以对象为中心的架构

这种架构中每个逻辑游戏对象会实现为类的实例，或一组互相连接的实例。然而单纯使用继承和多态会导致一系列类层次结构的问题。

### 1.1.1 使用面向对象架构的问题

类层次结构逐渐变得单一庞大。如下图①实现《吃豆人》（PacMan）的一种简单类结构，随着功能增长，该结构会同时往纵、横方向发展，并出现以下问题：

* 类难以理解、维护及修改：要理解一个类，就要理解其所有父类（例如在派生类中修改一个看似无害的虚函数，就可能会违背了众基类中某个基类的假设），参考下图②复杂的单一类树节选
* 不能表达多维分类：继承有着“是一个”的语义，导致在分类对象时只能从一个维度去设计。如下图③，各类载具的分类看似合乎逻辑，但如果再加入一种“水陆两用载具”则无从下手
* 多重继承的弊端：解决“水陆两用载具”的解决方法之一就是使用C++的多重继承，如下图④。然而多重继承有其严重弊端，此处不再赘述
* 使用接口：像C#或Java类只能继承一个类，但可以实现多个接口，这样共用的功能就能抽出来（也称为mix-in类）。如下图⑤，任何继承MHealth的类会有血量信息，并可以被杀
* **冒泡效应**：当游戏加入越来越多的功能，程序员很容易不断把若干个类中**公用但与基类无关**的代码上升到基类中（即为了所谓的复用利用了继承的便利），这种趋势会令功能代码沿层次结构上移到基类（冒泡），从而违背类职责应该保持单一的原则

![使用单一庞大的类层次结构的各种问题](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/engine/%E4%BD%BF%E7%94%A8%E5%8D%95%E4%B8%80%E5%BA%9E%E5%A4%A7%E7%9A%84%E7%B1%BB%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84%E7%9A%84%E5%90%84%E7%A7%8D%E9%97%AE%E9%A2%98.jpg)

### 1.1.2 使用“合成”来简化层次结构

面向对象设计中过度使用“是一个（is-a）”关系，会限制了我们创造新游戏类型的设计选择，而且难以扩展现存类的功能。若像下图左边的继承结构，希望一个游戏对象类有碰撞功能，它必须要继承自CollidableObject ，即使它可能是隐形的而并不需要RenderableObject的功能。若把不同的功能分离为独立的“组件”类，它们互不相干，由一个轻量的GameObject采用“有一个（has-a）”关系持有并管理，如下图右边，则可以大大简化。Unity便是运用这种思想的例子。

![使用组件合成游戏对象](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/engine/%E4%BD%BF%E7%94%A8%E7%BB%84%E4%BB%B6%E5%90%88%E6%88%90%E6%B8%B8%E6%88%8F%E5%AF%B9%E8%B1%A1.jpg)

对于GameObject管理其组件声明周期的具体实现，具体的做法是GameObject持有所有可能组件的指针并默认为空，而具体的游戏对象继承GameObject后，自行初始化所需的基本组件，并实现自己的特殊组件。但是当需要扩展新组件时，都要修改GameObject类，不符合开闭原则，因此更好的做法是以下这种GameObject持有Component列表的结构。

![使用通用组件的设计](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/engine/%E4%BD%BF%E7%94%A8%E9%80%9A%E7%94%A8%E7%BB%84%E4%BB%B6%E7%9A%84%E8%AE%BE%E8%AE%A1.jpg)

## 1.2 以属性为中心的架构

以对象为中心，会自然地关注对象属性和行为。以属性为中心，则是先定义所有属性，再为每个属性键表存储关联该属性的对象，*像数据库表就是这种设计*。

这种设计的优点是趋向更有效地使用内存，因为只需储存实际上用到的属性；也更容易使用数据驱动的方式来建模。最后是比以对象为中心的模型更加缓存友好，因为有些游戏硬件的内存存取成本远高于执行指令和运算。把数据连续储存于内存之中，能减少或消除缓存命中失败。这种数据布局方式称为**数组的结构（struct of array）**。以下代码展示了与传统 *结构的数组（array of struct）*的对比。

    static const U32 MAX_GAME_OBJECTS = 1024;
    // 传统结构的数组方式
    struct GameObject
    {
        U32 m_uniqueId;
        Vector m_pos;
        Quaternion m_rot;
    };
    GameObject g_AllGameObjects[MAX_GAME_OBJECTS];

    // 对缓存更友好的数组的结构方式
    struct AllGameObjects
    {
        U32 m_UniqueId[MAX_GAME_OBJECTS];
        Vector m_Pos[MAX_GAME_OBJECTS];
        Quaternion m_Rot[MAX_GAME_OBJECTS];
    }
    AllGameObjects g_allGameObjects;

这种设计的缺点是单凭凑齐一些细粒度的属性去实现一个大规模的行为，并非易事。这种系统也可能更难以除错，因为程序员不能一次性地把游戏对象拉到监视视窗中检查它的属性。

# 2. 对象引用与世界查询

## 2.1 对象引用方法

### 2.1.1 指针

每个游戏对象通常需要某种唯一标识符以便互相区分，并且能在运行时或工具方（世界编辑器）找到所需的对象，也可用该标识符作为对象间通信的目标。当通过查询找到一个游戏对象时，需要以某种方式引用它。C/C++中最常见的做法就是使用指针，因为指针是实现对象引用最快、最高效并最容易使用的方式。但使用指针很容易出现孤立对象、过时指针、无效指针等问题，所以开发引擎的团队制定严格的编程惯例，或使用安全的约束方法如智能指针。

智能指针是一个小型对象，行为与指针非常接近，但其扩展了规避原始C/C++指针所衍生的问题。关于智能指针可参考C++的一些高级书目，此处不赘述，仅建议**尽量不要在项目中尝试自己实现恶心的智能指针**，如果必须使用，尽量选用像Boost这样的成熟实现。

### 2.1.2 句柄

句柄就是某全局句柄表的整数索引，而句柄表则储存指向引用对象的指针。下图说明了此数据结构。

![句柄引用对象的实现方式及常见应用](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/engine/%E5%8F%A5%E6%9F%84%E5%BC%95%E7%94%A8%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F%E5%8F%8A%E5%B8%B8%E8%A7%81%E5%BA%94%E7%94%A8.jpg)

虽然句柄可以实现为原始整数，但句柄表的索引通常会包装成一个简单类，以提供更方便创建句柄和解引用的接口。以下是一种简单实现（省略其他与句柄无关的实现）。注意在。

    /* GameObject类储存了它的句柄索引，当要创建新句柄时就不用以地址搜寻句柄表了 */
    class GameObject
    {
    private:
        GameObjectId m_uniqueId;  // 对象唯一标识符
        U32 m_handleIndex;  // 供更快地创建句柄
        friend class GameObjectHandle;  // 让它访问id及索引
    public:
        GameObject()
        {
            m_uniqueId = AssignUniqueObjectId();
            m_handleIndex = FindFreeSlotInHandleTable();
        }
    }

    // 定义句柄表的大小，以及同时间的最大对象数目
    static const U32 MAX_GAME_OBJECTS = ...;
    // 全局句柄表，只是简单的数组，储存游戏对象指针
    static GameObject* g_apGameObject[MAX_GAME_OBJECTS];

    /* 句柄封装类 */
    class GameObjectHandle
    {
    private：
        U32 m_handleIndex;
        GameObjectId m_uniqueId;
    public:
        explicit GameObjectHandle(GameObject& object) :
            m_handleIndex(object.m_handleIndex),
            m_uniqueId(object.m_uniqueId) {}
        // 句柄解引用
        GameObject* ToObject() const
        {
            GameObject* pObject = g_apGameObject[m_handleIndex];
            if (pObject != NULL && pObject->m_uniqueId == m_uniqueId)
                return pObject;
            return NULL;
        }
    }

## 2.2 对象查询方法

取决于具体的游戏设计，开发者需要根据业务来查询不同种类的对象，例如找出玩家视线范围内的所有敌人角色，找出所有血量少于80%的可破坏游戏对象等等。游戏团队通常要判断，在游戏开发过程中哪些是可能最常用到的查询类型，并实现专用的数据结构加速查询。以下列举了一些可用于加速某类游戏对象查询的专门的数据结构。

* 以唯一标识符搜寻：游戏对象的指针或句柄可储存于以唯一标识符为键的散列表或二叉查找树
* 对合乎某条件的所有对象进行迭代：可预先以某种条件排序，并把结果储存在某个列表（例如不断维护一个在玩家某半径范围内的所有对象的列表来加速查询实现范围内的敌人）
* 搜寻抛射体路径或对某目标点视线内的所有对象：通常会利用碰撞系统实现，多数碰撞系统会提供一些极快的光线投射功能
* 搜寻某区域或半径范围内的所有对象：用一些空间散列数据结构去储存游戏对象，如四叉树、八叉树、kd树等等

参考文献：电子工业出版社《游戏引擎架构》第14.1、14.2、14.5节
