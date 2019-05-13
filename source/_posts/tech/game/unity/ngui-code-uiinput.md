title: NGUI源码学习——UIInput
date: 2017-06-01 22:33:44
categories:
- 技术
- 游戏开发
- Unity
- NGUI
tags:
- NGUI
- Unity
---
UIInput是NGUI的输入控件，其在PC端表现为一个文本框，而在手机端则会拉起系统的键盘。本文主要记录其基本用法和原理，以及一些重要的实现和工具方法。

【参考版本：NGUI 3.6.9】

<!-- more -->

# 1. 基本用法和原理

必须和一个UILabel联动使用，用于显示输入的文字。在手机上，实际上封装了对`UnityEngine.TouchScreenKeyboard mKeyboard`的引用用于控制操作系统的虚拟键盘，用MOBILE宏隔开（在非手机设备上用这个类会编译错误）。

UIInput在使用时一般还需要挂一个collider，以便响应UICamera的OnPress和OnDrag事件。这两个事件都是通过`GetCharUnderMouse()`来获取字符索引位置，并更新selectionStart和selectionEnd，没有其他逻辑。

# 2. 重要方法

- `Validate`：验证字符串（逐字符检查）并返回处理过的字符串。若设置了onValidate，则用它来验证，否则根据validation属性（有整形、浮点数、用户名等等选项）用内置的算法来检查字符。字符不合法则跳过该字符，最后根据characterLimit属性检查限制长度。
- `Update`：**当被选中时，才执行以下逻辑**
    - 等待mKeyboard.active
    - 在input被激活时mSelectMe记录帧数，确保下面一步【打开键盘】操作只会被执行一次
    - **打开键盘**：若在手机平台上，获取TouchScreenKeyboardType、InputType、当前值等并调用[TouchScreenKeyboard.Open](https://docs.unity3d.com/ScriptReference/TouchScreenKeyboard.Open.html)打开与操作系统相关的键盘；若不在手机平台，则开启[Input.imeCompositionMode](https://docs.unity3d.com/ScriptReference/Input-imeCompositionMode.html)。最后return。
    - **手机上获取输入字符**：手机上通过`TouchScreenKeyboard.text`来获取虚拟键盘中的字符，并调用Insert/DoBackspace方法【见下方具体算法】。接着再通过mKeyboard的API确定是否调用Submit方法。
    - **PC上获取输入字符**：一般通过`Input.inputString`来获取输入的字符，并调用Insert方法显示。当使用输入法时，会用到[Input.compositionString](https://docs.unity3d.com/ScriptReference/Input-compositionString.html)ime来更新用户已键入的字符并显示出来。（例如输出“呵呵”要输入“hehe”，后者即为ime）
    - **光标动画**：一个简单的时间判断，每0.5s切换一次mCaret的显示状态
- `UpdateLabel`：**核心方法**，更新label的显示，最终要显示的字符串存储在processed变量中
    - 若input值为空，则`processed = selected ? "" : mDefaultText;`
    - 若inputType为密码形式，则processed为相同字符数量的星号
    - 处理label为ClampContent且限制为1行的情况：在这种模式下，移动光标或选区超出范围会可以自动滚动字符（类似scrollView）。通过`label.CalculateOffsetToFit(processed)`计算label尺寸可以容纳下的字符偏移，再通过一些逻辑计算裁剪掉processed
    - 创建（更新）选区和光标：先代码创建一个2×2的白色纹理，赋给挂在label底下的（也是代码创建）Highlight和Caret（均挂了UITexture）。最后调用`label.PrintOverlay(start, end, mCaret.geometry, mHighlight.geometry, caretColor, selectionColor);`来绘制出选区和光标矩形，其中涉及到geometry添加顶点，算法逻辑较为复杂。

# 3. 工具方法

- `Insert(string text)`：先获取选取当前选区（或光标）左侧串left和右侧串right，再遍历text的每个字符，处理退格、characterLimit和validate的情况。最后还要再对right进行validate**（防止插入的text破坏了后面的合法性）**
- `DoBackspace()`：原理是`Insert("")`，最终所得字符串为当前selection左侧字符串+空串+右侧字符串，等价于删掉了selection所在的字符，方法非常巧妙
- `ProcessEvent(Event ev)`：处理各种特殊按键事件，如箭头、退格、回车等等，绝大多数都是mSelectionStart和mSelectionEnd的逻辑处理，算法较为简单
