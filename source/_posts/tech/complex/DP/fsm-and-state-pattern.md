title: 状态模式与状态机
date: 2017-01-30 21:42:47
categories:
- 技术
- 综合
- 设计模式
tags:
- 状态模式
- 状态机
- 设计模式
---

状态管理是几乎所有游戏开发都会做的一个事情——大到场景管理，小到角色的逻辑状态和动画切换。如果你面对日益增长的复杂的状态分支判断代码而不知所措，状态模式和状态机可能就是救星。状态模式是一个非常简单实用的模式，本文将介绍其基本用法，并结合游戏中的有限状态机（FSM）介绍其各种实际应用和特殊状态机。

<!-- more -->

# 1. 有限状态机

状态在游戏里面非常常见。想象一个横版过关游戏的主角有如下图状态转换。

![一个简单的游戏角色状态例子](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/engine/%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84%E6%B8%B8%E6%88%8F%E8%A7%92%E8%89%B2%E7%8A%B6%E6%80%81%E4%BE%8B%E5%AD%90.jpg)

FSM有以下特性：

- 拥有一组状态，它们之间可以互相切换
- 状态机同一时刻只能处于一种状态（事实上防止同时存在两个状态正是使用FSM的原因）
- 状态机会接收一组输入或事件
- 每个状态有一组转换，每一个转换都关联一个输入并指向另一个状态

最普通（也是不可取）的做法，是*先用if/else分支来判断输入，再用各种bool变量来标记当前是否处于某种状态*。随着功能和状态的增加，会不经意破坏已有代码的功能，并引入更多的标记变量使逻辑更加复杂。

更进一步的做法是将每种状态定义为枚举，颠倒一下上面的处理顺序，*先用switch/case判断状态，再在状态中判断输入并做状态切换*。如果状态非常少而且不太可能扩展，这是实现状态机最简单有效的做法。

当问题满足以下几点要求时，FSM将会非常有用：

- 一个实体的行为基于它内部状态而改变
- 这些状态被严格划分为数量较少的小集合
- 实体随着时间变化会响应用户输入或一些特殊事件

# 2. 状态模式

状态模式实际上就是**将复杂的各个分支判断分离出来成为一个个单独的状态类（继承同个状态基类），然后在业务类中维护状态基类指针m_state，通过多态来分发和切换**。

为了修改一个状态，之前的做法是给m_state赋值新的枚举值（或常量数字），而将状态抽象为类之后，有两种做法：

- 静态状态：如果一个状态对象没有数据成员，可以在状态基类（或其他地方）定义static状态对象。【进一步的，若状态仅包含一个虚函数方法，可以用状态函数来替代状态类，这样m_state就编程函数指针】
- 实例化状态：若同类的状态会被多个对象维护，就不能使用静态状态了，只能在状态切换时动态创建实例（要留意清理老的状态实例）。此外，在状态类的处理虚函数中，可以返回一个新的状态实例表示下一个要切换的状态（返回null则状态不变），这样业务类就能根据返回的结果修改m_state的值。

# 3. 特殊种类的状态机

## 3.1 并发状态机

设想一个角色有跑、跳、躲避等状态，现在要添加持枪功能（包括不开火和开火两种状态）。如果执着于传统的FSM，则可能需要跑动和跑动开火等n*m种状态。如果使用并行的两个状态机，则只需要n+m种状态。

具体实现上，只需要定义多一个状态成员变量，在响应事件多派发一次即可。这种实现适用于两个状态互相独立。若两个状态有依赖，例如不能在跳跃过程中开火，则可能需要在特定的状态中做一些简单的if判断，必要时还需要引用另一个状态。

## 3.2 层次状态机

假设一个角色无论处于站立、走路、跑动、滑动哪个状态，按下B时都要跳跃，按下↓方向键都要躲避。若只用简单的FSM，则会重复不少代码（比如这个例子需要处理4*2个状态转换）。可以在状态模式的基础上，采用继承的方式来复用代码，例如定义OnGroundState来处理跳跃和躲避的状态，四种状态从OnGroundState继承。当有一个事件进来时，如果子状态不处理，则将事件传递给父状态处理。

不过要注意，这种方法**仅仅只是为了复用代码而去继承，并不符合继承的语义**，像上例中WalkState“是”（is-a）一种OnGroundState吗？因此选用这种“继承层次”的状态机要慎重。

## 3.3 下推自动机

这种状态机使用一个状态**栈**，来解决FSM没有历史记录的问题。举个例子，角色可以从任一状态切换到开火状态，该状态会播放动画，发射子弹并显示一些特效，合理来说开火完成之后应该回到之前的状态。若只用简单的FSM，则需要为每个状态定义一个反向的转换。而使用状态栈时，通过push和pop来记录新状态和恢复历史状态。记住当前状态永远在栈顶，如果不需要历史记录，则切换状态时，总把栈顶的状态修改为最新的状态即可。

参考文献：《游戏编程模式》第7章
