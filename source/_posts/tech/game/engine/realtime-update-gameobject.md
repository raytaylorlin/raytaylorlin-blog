title: 运行时游戏对象模型（下）——实时更新游戏对象
date: 2017-01-13 21:33:32
categories:
- 技术
- 游戏开发
- 游戏引擎
tags:
- 游戏引擎
- 游戏对象模型
---
大多数低阶引擎子系统，以及游戏对象，都需要周期性更新。差不多所有游戏引擎都在主游戏循环里更新游戏对象的状态，换句话说，它们把游戏对象模型当作另一个需要周期性运行的引擎子系统。本文将简要介绍关于实时更新游戏对象的设计方式及常见问题。

<!-- more -->

# 1. 实时更新对象的方式

一种最简单但**不可行**的实现方式是，每个游戏对象都有一个虚函数`virtual void Update(float dt)`，游戏主循环在每一帧遍历全体游戏对象集合并逐一调用Update。每个Update所做的事情大致是更新对象自身的逻辑数据，然后逐个更新其组件（如动画、渲染、粒子、声音组件）。

## 1.1 性能限制与批次式更新

低阶引擎系统都有极严竣的性能限制，**把多个游戏对象的同个子系统更新组合起来批次处理**，要比上述*多个游戏对象交错更新子系统*更高效，如下图所示。像渲染引擎就是使用批次式更新的典型例子。

![交错式更新与批次式更新的区别](http://raytaylorlin-blog.qiniudn.com/image/engine/%E4%BA%A4%E9%94%99%E5%BC%8F%E6%9B%B4%E6%96%B0%E4%B8%8E%E6%89%B9%E6%AC%A1%E5%BC%8F%E6%9B%B4%E6%96%B0%E7%9A%84%E5%8C%BA%E5%88%AB.jpg)

批次式更新带来很多性能效益，包括但不限于：
* 最高的缓存一致性：子系统能把各对象的所需数据分配到一个连续的内存区里
* 最少的重复运算：可以先执行整体的运算，之后在各对象更新中重用，无须每次在对象中重新计算
* 减少资源再分配：交错式更新处理每个对象时须释放及再分配资源，批次式更新则只需每批次一次
* 高效的流水线：在某些硬件上可以做一些优化，利用硬件特设的资源并行计算

性能优势并不是使用批次式更新的唯一原因，**一些引擎子系统从根本上不能以对象单位进行更新**。例如，若一个动力学系统里有多个刚体进行碰撞决议时，孤立地逐一考虑对象，一般不能找到满意的解。

## 1.2 对象及子系统的相互依赖

要正确运行游戏，游戏对象更新的次序是重要的（例如计算某物体的局部坐标需要先计算其父节点的世界坐标）。除了对象之间有依赖关系，各子系统也有依赖关系，而且不是简单的先后关系，例如布娃娃物理模拟系统须与动画系统协同更新。可以在主循环中明确编写各个子系统的更新顺序。

主循环通常不能简化成每帧每对象调用一次Update，游戏对象可能需要使用多个引擎子系统的中间结果。很多游戏引擎容许游戏对象在1帧中的多个时机编写对应的虚函数“挂钩”进行更新，像Unity GameObject的Update、FixedUpdate、LateUpdate等。游戏对象可按需增加更多更新阶段，但要小心带来多余的调用空的虚函数开销可能很高。

## 1.3 桶式更新

当存在对象间的依赖时，可能会抵触更新次序的规则，有时要轻微调整上述的批次式更新技巧。即不要一次性批处理所有游戏对象，而是把对象按依赖关系分为若干群组（或称为桶bucket），即没有任何依赖关系的对象（依赖树的根）放到第1个桶，依赖树第2层的所有对象放到第2个桶……然后按依赖次序更新每个桶，桶中使用批次式更新，如下图所示。游戏引擎可以明确为依赖树林的深度设限，这样就可以使用固定数目的桶以提高性能。

![按对象的依赖性分桶更新](http://raytaylorlin-blog.qiniudn.com/image/engine/%E6%8C%89%E5%AF%B9%E8%B1%A1%E7%9A%84%E4%BE%9D%E8%B5%96%E6%80%A7%E5%88%86%E6%A1%B6%E6%9B%B4%E6%96%B0.jpg)

# 2. 对象状态及“差一帧”延迟

更新游戏对象可视为这样一个过程：每个对象根据$t_1$时刻的状态决定$t_2$（$t_2=t_1+\Delta t$）时刻的状态。理论上，所有游戏对象的状态是瞬间及并行地从时刻$t_1$更新至$t_2$的。但实际上主循环会逐个更新对象，在一轮循环中间中断时则有一些对象处于部分更新的状态（例如某个对象可能已执行姿势动画混合，却未计算物理及碰撞决议）。

游戏对象在两帧之间状态不一致是混淆和bug的主要来源。当有对象依赖时（如对象B需要根据对象A的速度来决定当前帧自身的速度），程序员必须弄清楚需要的是对象A的之前的状态还是新状态。若需要新状态，而对象A却未更新，就会产生一个更新次序问题，会导致一类称为“差一帧”延迟的bug。解决这个问题通常有以下做法：

* 桶式更新：上节已描述，但是必须保证同一个桶内的对象不会互相查询状态
* 对象状态缓存：更新时不要就地覆写新的状态，而是保留之前的状态变量，并把新的状态写到另一个变量。这样任何对象都可安全地查询其他对象的之前状态；而且就算是在更新的过程中，它保证永远有一个完全一致的状态；还能通过线性地向前后两个状态插值。这种方法的缺点是多耗一倍内存，而且只能保证在$t_1$状态一致，而$t_2$状态不一定一致
* 加上时戳：给每个对象加时戳可轻易分辨对象的状态是在之前还是当前时间

参考文献：电子工业出版社《游戏引擎架构》第14.6节