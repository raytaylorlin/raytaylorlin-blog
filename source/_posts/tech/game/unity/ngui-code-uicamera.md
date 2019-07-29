title: NGUI源码学习——UICamera
date: 2017-03-10 22:02:07
categories:
- 技术
- 游戏开发
- Unity
- NGUI
tags:
- NGUI
- Unity
---
UICamera是NGUI中专门用于捕获和分发交互事件的脚本，和UI渲染无关，需要挂在UI摄像机上。其核心思想是在Update中检测Input的各种输入情况，并对屏幕做raycast投射，以决定是哪个go的collider触发事件，最终将事件分发出去。

【参考版本：NGUI 3.8.1】

<!-- more -->

# 1. 核心数据结构或类型

有许多公共静态属性，如currentXXX，用于存放当前触发事件的摄像机、ray等等。

- EventType：分World_3D、UI_3D、World_2D、UI_2D，其中3D用`Physics.Raycast`实现，2D用`Physics.OverlapPoint`实现；World会根据触发点的世界距离排序（常用于游戏摄像机），UI会根据widget depth排序（常用于UI界面）
- `enum ControlScheme`：有鼠标、触摸、手柄三种类型，触发事件时会根据这个类型做相应的调整（如hover或selected事件的分发在不同设备是不一样的）
- MouseOrTouch数据结构：在事件触发之前会设置一些鼠标或触摸信息


    public class MouseOrTouch
    {
        public Vector2 pos;               // 当前鼠标或触摸的位置
        public Vector2 lastPos;           // 上一次鼠标或触摸的位置
        public Vector2 delta;             // 当前帧与上一帧的偏移
        public Vector2 totalDelta;        // delta的累积，通常用于drag事件

        public Camera pressedCam;         // OnPress(true)触发时对应的捕获事件的摄像机

        public GameObject last;           // 上一个触发触摸或鼠标事件的go
        public GameObject current;        // 当前触发触摸或鼠标事件的go
        public GameObject pressed;        // 上一个接收OnPress的go
        public GameObject dragged;        // 正在被拖拽的go

        public float clickTime = 0f;      // 上一次click事件的时间（通常用于判断doubleClick）

        public ClickNotification clickNotification = ClickNotification.Always;  // OnClick的触发条件，None为不触发，Always为总是触发，BasedOnDelta为根据位置移动的偏移量来决定是否发生（偏移量和Thresholds参数有关）
        public bool touchBegan = true;    // Touch模式下标识一个触摸是否为开始
        public bool pressStarted = false; // 标识是否开始按住
        public bool dragStarted = false;  // 标识是否开始拖拽
    }

下图为UICamera脚本的配置。

![UICamera脚本的配置](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/unity/UICamera%E8%84%9A%E6%9C%AC%E7%9A%84%E9%85%8D%E7%BD%AE.jpg)

# 2. 核心方法

## 2.1 Notify(GameObject go, string funcName, object obj)

本质上是调用`go.SendMessage(funcName, obj)`，这样就能**触发到具体的UI控件中与funcName同名的方法**（正因为是通过这种特殊方式来调用，所以控件源码中一部分方法会查找不到引用，和反射的道理类似）。

同时也会发一份消息到genericEventHandler这个用户自己设置的全局go，相当于一个全局的事件接收器。

## 2.2 Update

本质上是对UnityEngine.Input的封装和处理。

1. 根据useTouch或useMouse标记（在Awake中根据当前平台设置，如手机只有touch），选择执行`ProcessTouches`或`ProcessMouse`
2. 调用用户自定义的委托`onCustomInput`
3. `ProcessOthers`处理键盘和手柄
4. 处理tooltip相关逻辑

以`ProcessTouches`为例，该方法中用`Input.GetTouch`获取每个touch分别做如下处理：

- 创建MouseOrTouch结构并设置数据
- 调用`ProcessTouch`分发事件
- 若touch数目为0，则转为`ProcessMouse`或`ProcessFakeTouches`（用于编辑器用鼠标模拟触摸）
- `ProcessTouch`：根据传入的pressed，向`currentTouch.pressed`分发OnPress事件；根据`currentTouch.delta`或touch前后go是否不同，向`currentTouch.dragged`和`currentTouch.last`分发OnDragStart和OnDragOver事件；后续还有一段处理各种drag start、over、out的逻辑。根据传入的unpressed，分发OnClick、OnSelect、OnHover等事件。
- `ProcessOthers`：处理submit（如回车和手柄的按键）、方向键、返回键、tab键的情况，并对mCurrentSelection分发OnKey事件

## 2.3 RayCast（Vector3 inPos）

给定位置判断有没有与控件产生交互，最终要得到hoveredObject这个结果。

- `currentCamera.ScreenToViewportPoint(inPos)`算出viewport的坐标，并排除一些异常情况
    - 【注：屏幕坐标左下角是(0,0)，右上角是(pixelWidth,pixelHeight)】，viewport坐标右上角是(1,1)】
- `currentCamera.ScreenPointToRay(inPos)`将屏幕坐标转换为ray。UICamera有个表示射线长度的参数rangeDistance，默认为摄像机远近裁剪面的距离；eventReceiverMask表示摄像机投射ray时哪些层可以响应
- 接下来根据EventType采用不同的算法来算ray射到的物体，以两种3D模式为例：
    - World_3D：`if (Physics.Raycast(ray, out lastHit, dist, mask)) hoveredObject = lastHit.collider.gameObject`
    - UI_3D：`Physics.RaycastAll(ray, dist, mask)`获取射线穿到的所有hit，取每个hit对应的collider的go，计算其raycastDepth，并按从大到小排序
    - hoveredObject = 上述最大的，且对应panel可见的hit对应的go

【定义：UIWidget.raycastDepth = 自身depth + 所属panel.depth * 1000】
【定义：`NGUITools.CalculateRaycastDepth(go)`计算go下所有可用widget的raycastDepth的最小值】

参考文献：[Yarpee的博文《UICamera》](http://www.yarpee.com/?p=150)
