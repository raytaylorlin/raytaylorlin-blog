title: 《看门狗》网络绕接玩法的实现
date: 2018-02-10 16:17:53
categories:
- 技术
- 综合
- 项目
tags:
- 看门狗
---
《看门狗》系列游戏中，网络绕接（Network Bypass）是其中一种解谜小玩法，其目标是通过旋转一些通路节点来打通通路，最终解锁目标节点。下图展示了一个游戏中的示例，看一遍应该就能明白。

<video src="https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/project/%E3%80%8A%E7%9C%8B%E9%97%A8%E7%8B%97%E3%80%8B%E7%BD%91%E7%BB%9C%E7%BB%95%E6%8E%A5%E7%8E%A9%E6%B3%95%E7%A4%BA%E4%BE%8B.mp4" type="video/mp4" controls="controls" width="100%" height="100%"></video>

鉴于这个玩法非常简单，闲暇时正好拿来作为练手项目。本文描述了该项目大致的设计思路。[Github项目地址](https://github.com/raytaylorlin/NetworkBypassPuzzle)

<!-- more -->

# 1. 设计
## 1.1 节点类型

- 起始节点（StartNode）：每个网络绕接玩法至少会有一个，这是所有网路通路的开端
- 终止节点（EndNode）：每个网络绕接玩法至少会有一个，且必定带锁，玩法的目标就是解开这个节点的锁
- 连接节点（ConnectionNode）：起到变向网络流的功能，给定一个方向的输入，会有其他方向的输出
    - 直角型节点（RightAngleConnectionNode）
    - 直线型节点（StraightConnectionNode）
    - T型节点（TConnectionNode）
    - 十字型节点（CrossConnectionNode）
- 转接节点（TransitionNode）：用于改变网络流的流动方向

## 1.2 节点共性

下面一级列表列出各种节点的共同特性（父类）。一些特殊节点（子类）的具体实现方式列在二级列表。

- 每个节点都有上下左右4个邻居节点（可空）
- 连接节点有4种旋转方向
    - 像十字型无论怎么转都一样，直线型只有两种方向，但4个旋转方向对它们来说依旧适用
- 4个方向都可能有输入和输出
    - Start节点只有输出
    - End节点只有输入
    - 连接节点根据自身旋转方向和输入方向，来确定朝哪个方向输出
- 都可以被激活或反激活
    - 当连接节点的某个管道有输入时，即被激活
    - 带锁节点（包括终止节点）的所有锁口有输入时，即被激活
    - 当转接节点某个方向有输入时，即被激活

## 1.3 装饰器

装饰器是用来给节点添加额外功能的，最常用的就是加锁（也是项目目前实现的唯一一种装饰器）。在实际的《看门狗》游戏中，还存在两种装饰器：倒计时装饰器（所装饰的节点被激活时开始倒计时，计时结束则重置整个网络绕接谜题，强迫玩家在限定时间内完成谜题），打乱装饰器（当某种条件达成时，例如某个节点被激活，则打乱所装饰节点的旋转方向，起到一种干扰作用）。

锁装饰器对于连接节点是可选的，但对于终止节点是强制要有的。锁也是定义了4个方向，当没有解锁之前，上了锁的节点会阻断其他输入流，当所有锁都解开时，其原本的节点功能才会恢复。

利用Unity天然的组件系统，各种装饰器可以轻松地做成Component并附加在节点上。

# 2. 实现
## 2.1 核心算法

网络绕接的实现可以说是一个遍历有向图的过程。核心思想是从起始节点出发，沿着网络流广度优先遍历图的边。算法的伪代码如下：


    void Init()
    {
        遍历所有节点，在邻居节点之间创建NetworkFlow（边）;
        初始化所有节点，并注册相关事件;
    }

    private Queue<NetworkFlow> flowQueue = new Queue<NetworkFlow>();
    // 遍历网络流，在玩法开始时执行一次，随后每次节点执行一次操作则再执行该方法
    void TraverseNetworkFlow()
    {
        将所有的NetworkNode和NetworkFlow都清理一下（反激活）;
        将起始节点周边有输出的通路，入队flowQueue;
        while (flowQueue.Count > 0)
        {
            NetworkFlow flow = flowQueue.Dequeue();  // 当前处理的网络流
            NetworkNode node = flow.FirstNode;       // node可理解为当前处理的节点
            NetworkNode neighbor = flow.SecondNode;
            激活flow;
            设置neighbor的输入状态;

            // 第一趟查找
            将neighbor周边有输出的**没有被访问过的**通路，**而且该通路从来没有走过**，则入队flowQueue;

            if (第一趟查找没有找到任何flow)
            {
                将neighbor周边有输出的通路，**而且仅被访问过一次（说明这一次是的流向是反向的）**，入队flowQueue;
            }
        }
    }


## 2.2 谜题编辑器

要实现这个玩法规则本身并不是很难，但此外还有一个要重点考虑的是如何让策划高效地编辑和测试各种谜题。为此特意对各种节点类型定制了Inspector，如下图所示，其作用应该也是一目了然。

![网络绕接玩法：节点定制的Inspector](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/project/%E7%BD%91%E7%BB%9C%E7%BB%95%E6%8E%A5%E7%8E%A9%E6%B3%95%EF%BC%9A%E8%8A%82%E7%82%B9%E5%AE%9A%E5%88%B6%E7%9A%84Inspector.jpg)

下面的示意图演示了如何创建节点，摆放节点，以及给节点连线和设置参数。

![网络绕接玩法：创建，摆放节点](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/project/%E7%BD%91%E7%BB%9C%E7%BB%95%E6%8E%A5%E7%8E%A9%E6%B3%95%EF%BC%9A%E5%88%9B%E5%BB%BA%EF%BC%8C%E6%91%86%E6%94%BE%E8%8A%82%E7%82%B9.gif)

![网络绕接玩法：给节点连线和设置参数](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/project/%E7%BD%91%E7%BB%9C%E7%BB%95%E6%8E%A5%E7%8E%A9%E6%B3%95%EF%BC%9A%E7%BB%99%E8%8A%82%E7%82%B9%E8%BF%9E%E7%BA%BF%E5%92%8C%E8%AE%BE%E7%BD%AE%E5%8F%82%E6%95%B0.gif)

2.3 网络通路的噪声效果
最开始的实现是直接用一个Line Renderer实现的。后来尝试做出原版游戏的那种噪声效果，在网上找了一个随机生成雪花点的Shader，并自己修改成一个Shader动画，其Pass代码如下：


    CGPROGRAM
    #pragma vertex vert_img
    #pragma fragment frag

    #include "UnityCG.cginc"

    // 这三个参数影响雪花点的分布，经我测试三个均使用5000可达到比较好的效果
    float _Factor1;
    float _Factor2;
    float _Factor3;
    // 雪花点闪烁的速度
    float _Speed;
    fixed4 _Color;

    float noise(half2 uv)
    {
        float t = _Speed * _Time.y;
        return frac(sin(dot(uv, float2(t * _Factor1, t * _Factor2))) * t * _Factor3);
    }

    fixed4 frag (v2f_img i) : SV_Target
    {
        fixed4 col = noise(i.uv);
        col.rgb *= _Color.rgb;
        return col;
    }
    ENDCG


最终效果如下：

![噪声Shader动画](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/project/%E5%99%AA%E5%A3%B0Shader%E5%8A%A8%E7%94%BB.gif)

最后放一个比较复杂的且支持3D的案例演示：

<video src="https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/project/%E7%BD%91%E7%BB%9C%E7%BB%95%E6%8E%A5%E7%8E%A9%E6%B3%95%EF%BC%9A%E5%A4%8D%E6%9D%82%E7%9A%843D%E7%A4%BA%E4%BE%8B.mp4" type="video/mp4" controls="controls" width="100%" height="100%"></video>