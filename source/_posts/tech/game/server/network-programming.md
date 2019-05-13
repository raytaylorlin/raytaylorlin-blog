title: 网络游戏编程基础知识
date: 2015-11-11 10:27:40
categories:
- 技术
- 游戏开发
- 服务器端
tags:
- 网络游戏编程
---
本文首先介绍网络游戏开发者需了解的网络编程基础，包括对应的OSI七层分层模型，与游戏架构的关系。然后介绍套接字API及RPC相关的内容。

<!-- more -->

# 1. 开发者需了解的网络编程基础

## 1.1 网络游戏对应的OSI模型

![网络游戏对应的OSI模型](http://raytaylorlin-blog.qiniudn.com/image/server/网络游戏对应的OSI模型.png)

一般来说，第4层以下的分层，交由操作系统来处理即可。第4层大多使用TCP，只有在有必要的情况下才使用UDP（例如**发送那些与可靠性相比到达速度更为重要的数据**（如FPS游戏），实现NAT遍历功能等等）。由于游戏类型和策划内容千差万别无法统一，第5层及以上的功能需要网游开发人员自己实现。

## 1.2 网络编程特性与游戏架构的关系

1. C/S架构游戏（C/S MMO、C/S MO）：高性能高功能服务器端编程+一般程度的客户端编程
2. P2P架构游戏（P2P MO）：一般程度的服务器编程+高性能高功能呢的客户端编程（因为客户端要扮演服务器的角色）

高性能高功能服务器的特性：

1. 小带宽：每秒几次至20几次，达到几百位通信量的持续连接
2. 极高的连接数：每台服务器需要维持数千至数万个连接
3. 低延迟：处理并返回结果的延迟，只能在几毫秒至20毫秒以内
4. 稳定：服务器端保持游戏状态、敌人等可以移动的物体实时地持续行动

与服务器端相比，客户端的连接数较少，但是需要进行渲染等重要处理，还必须在延迟很低的情况下进行通信，并应对网络状况的多样性（如防火墙、各ISP的策略间差异等等）。

# 2. 套接字API

## 2.1 网络游戏中的套接字API

[BSD套接字API](https://zh.wikipedia.org/wiki/Berkeley%E5%A5%97%E6%8E%A5%E5%AD%97)（即Scoket API）是为了实现互联网连接而开发的API，是在所有操作系统上进行网络开发的首选。关于套接字API编程基础可见WIKI及网上各种资料。

使用第4层的套接字API，可以在不具可靠性的IP协议上实现两种类型的通信：一种是**面向连接的流式（Stream）通信**，在简历了连接的两台主机间维持通信线路畅通，保证通信持续进行；另一种是**无连接的数据报（DGram）通信**，只进行一次数据报交换，不维持主机间的通信线路。

套接字API中的`accept()`函数在“新的连接请求到来前一直等待着”，显然不能满足网络游戏服务器为多个客户端同时提供服务的要求。为了解决这个问题必须处理多个并发连接，方法大致有：

1. 每次连接时启动一个进程：**不可用**，因为网络游戏中需要多个用户连接实时共享同一个游戏状态
2. 使用线程并行进行同步处理：**不可用**，几千个连接启动几千个线程会使服务器性能大幅下降
3. 异步多重输入输出：使用`select`函数事先查询所带的消息（数据及连接请求）是否已经到达，即轮询。（使用`poll`及更高速的`epoll`函数也可实现同样的功能

网络游戏编程中同时处理数千个可移动物体是很平常的，因此客户端和服务器端通常都使用select（或poll/epoll）在**单线程**中实现简单的**事件驱动**的**非阻塞**模式。通过这种模式，还可以充分发挥出**多核**服务器的性能。

实现服务器端的最佳程序库是[libevent](http://libevent.org/)，这是一个跨平台的基于事件和回调的库，全世界应用广泛，不管是性能还是稳定性都比较成熟。

## 2.2 多核处理器与网络吞吐量

服务器通常用以太网连接至数据中心的网络中，通信速度为1Gbit。但是网游中经常会发送大量的小数据包，由于以太网在发送IP数据包时会向数据包中添加IP数据意外的信息一起发送，所以实际上应用程序能够使用的带宽要更小。

根据经验，将理论值的1/10作为基准，1Gbit/s以太网每秒可以发送100MB的数据，能够发送的数据包最好以每秒10W-15W为上限。如果在有10个内核的机器使用1Gbit/s以太网，每个内核可以处理大约1W个数据包，若同时连接数为每个内核1000个连接，则每个连接必须设计为发送频率限制在10次/s以内；或者，安装多个网络适配器，连接4根LAN电缆来实现4倍的吞吐量。

# 3. RPC通信中间件

远程过程调用协议RPC（Remote Procedure Call），将与通信有关的一些复杂细节封装起来，与一般的函数调用形式相同，是确保与远程主机进行简单、安全通信的一种方法。有了RPC，就不需要直接使用复杂的套接字API进行网络编程了。

## 3.1 通信库的必要性

单纯使用套接字API之所以会很复杂，是因为会根据网络状况产生这些问题：不一定能成功收发期望数据，之后需要再次调用；可能会发生错误；发送缓存满了的话，write()函数会等待；发送了不完整的内容。

套接字API中的send在发送成功前不会阻塞，每次编写错误处理造成的代码重复也是引起很多错误的根源。因此需要一个能独自负责这些工作的程序库，这个库应首先针对网络的IO要求装入缓存中，接着准确地执行，再将数据发送出去直至完成，若一段时间内无法发送则返回错误信息。总而言之，通信库会对诸如`send`这样的函数进行封装，并确定像`[数据类型代码][数据内容]`的数据格式，来收发数据。

## 3.2 网游中使用的RPC整体结构

RPC的基本原理是在本地模拟远程主机的函数调用，主要通过将数据流进行编码后发送出去，远程主机接收数据并解码，然后调用相应的函数。下图展示了网游中RPC的基本模式。注意到调用侧应用程序调用了`attackAtEnemy`函数，该函数定义在源文件“RPC存根代码”中，存根代码是用工具自动生成的，不需要手工编写。其中“123”固定值表示要调用`attackAtEnemy`这个函数，“99”表示要攻击id为99的敌人。

![网络游戏中使用的RPC模式](http://raytaylorlin-blog.qiniudn.com/image/server/网络游戏中使用的RPC模式.jpg)

RPC存根代码文件中调用方和被调用方的函数参数列表必须完全一致，如果有大量函数，应该采用RPC工具来自动生成。通常使用Ruby或Python等很容易进行DSL（领域特定语言）定义的语言来设计[IDL（接口描述语言）](https://zh.wikipedia.org/wiki/%E6%8E%A5%E5%8F%A3%E6%8F%8F%E8%BF%B0%E8%AF%AD%E8%A8%80)，然后执行脚本生成存根函数的源代码和头文件。

# 4. 确保开发效率和可移植性

* 正式服务器采用Linux，但开发环境则是在Windows下用Visual Studio以高效地开发
* 服务器端和客户端在碰撞检测等方面使用相同的游戏处理代码，确保可移植性

为了降低操作系统的差异性，需要对以下这些基础API进行封装以保持可移植性：

* 内存管理：malloc几乎在所有的操作系统中都会使用，所以很容易封装
* 套接字API：Windows和UNIX系统（包括iOS）有所不同
* 线程：封装pthread的基本API即可
* 信号：远程管理服务器的情况下需要使用信号，但这是一种可移植性很低的方法，并不推荐
* 事件与计时：使用libevent

网络编程中，对所有套接字调用select函数进行轮询，对于需要处理的内容执行read和write操作，调用回调函数来逐个处理；在客户端游戏编程中，对所有可移动物体以帧为单位进行轮询，对于需要进行处理的物体调用回调函数来使其行动。因此，无论是服务器端还是客户端，大多使用**单线程**来完成开发。

参考文献：人民邮电出版社《网络游戏核心技术与实战》第0章