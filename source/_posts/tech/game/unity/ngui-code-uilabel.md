title: NGUI源码学习——UILabel
date: 2017-04-5 22:02:07
categories:
- 技术
- 游戏开发
- Unity
- NGUI
tags:
- NGUI
- Unity
---
UILabel是NGUI中用于显示文字的控件。其核心思想是遍历每个字符，将其转化为字体贴图的数据。本篇会抛开BBCode解析，表情符symbol解析，特殊效果（阴影、描边等）等细节，只记录几个渲染字体的核心方法。NGUI有独立的一套BMFont管理工具和类，等以后再另开篇章讲解。

【参考版本：NGUI 3.6.9】

<!-- more -->

# 1. 基础要点

- 基本上只要设置了某个属性，都会使shouldBeProcessed变成true。在必要的时候，例如设置text、fontSize、fontStyle、alignment、OnValidate()，或者在获取父类UIWidget的一些属性，如localSize、drawingDimensions、worldCorners等等，才会根据shouldBeProcessed去调用`ProcessText`重绘字体。
- NGUIText作为中转处理的静态类，包含了大量的公共静态属性供缓存
- 字体方案有两种：Unity自带的动态字体（TTF）和NGUI的BitmapFont（即图字[BMFont](http://www.angelcode.com/products/bmfont/)）。后者在获取glyph信息时会多考虑当前字符与上一个字符的字距。

# 2. 核心方法

## 2.1 ProcessText(bool legacyMode, bool full)

- `NGUIText.Update(false)`：计算最终字体尺寸、space大小、行高
- `NGUIText.WrapText`：根据rectHeight、maxLines、finalLineHeight算出在widget最多可以容纳多少行。遍历每个字符，依旧获取glyph信息，并维护剩余宽度remainingWidth来决定是否截断显示字符串。该方法还会对空格和换行做一些特殊处理。最后输出fits表示rect是否能容纳得下应该显示的字符串，finalText表示最终显示的字符串
- `NGUIText.CalculatePrintedSize(mProcessedText)`：计算最终字符显示的区域大小，结果可从printedSize属性获取（小于等于rect大小）

几种Overflow模式的基本原理：

- ShrinkContent缩放原理：循环从ps = mPrintedSize = defaultFontSize开始往下递减，直到ps可以满足在当前rect里容纳显示所有字符。例如当一个240×30的label不能一行显示字号为60的“New\nlabel”，则ps可能会递减到30左右，`NGUIText.WrapText`计算出可以fit整个rect了，才停止循环。
- ClampContent裁切原理：实际上这种方式没有做任何处理，因为在`NGUIText.WrapText`的时候就已经按rect大小做截断了
- ResizeFreely自适应原理：一开始先把rect和region设得足够大，WrapText之后再重算printedSize
- ResizeHeight：同ResizeFreely，但只会重设高度，不会重设宽度

## 2.2 OnFill(BetterList<Vector3> verts, BetterList<Vector2> uvs, BetterList<Color32> cols)

在UIWidget调用UpdateGeometry时用到。

- `UpdateNGUIText`：将label的各种属性复制到NGUIText的公共静态属性中
- `NGUIText.Print(text, verts, uvs, cols)`将文本内容输出到列表中。
- `ApplyOffset`用偏移值变换顶点：原理是根据pivotOffset（锚点相对偏移，右上角为(1,1)）和label的宽高插值出offset，然后应用到顶点列表的每个顶点，最后返回offset
- 如果有开启阴影，则调用一次ApplyShadow【增加一些顶点】；如果开启描边，则再调用3次ApplyShadow，实质上就是用四个方向的阴影包围来模拟描边

## 2.3 NGUIText.Print(text, verts, uvs, cols)

- `Prepare(text)`：当使用动态字体时，调用Unity API `Font.RequestCharactersInTexture`刷新所需字符的纹理
- 遍历text的每个字符 =>
- 处理换行符号，略过非法字符
- `ParseSymbol`解析BBCode，该函数有很多ref参数，用于存放解析结果（如加粗、斜体、下划线等等）
- `GetSymbol(text, i, textLength)`获取有没有图字符号（新版本NGUI label支持表情符解析），有符号走符号分支，否则进入普通字符分支。【下面只讲解普通字符分支】
- 处理alignment为居中或右侧的情况

普通字符处理：

- `GetGlyph(ch, prev)`：根据当前字符ch和上一个字符prev，会根据使用的是位图字体还是动态字体，计算出对应的UV坐标。【得到的GlyphInfo数据结构见下方解释】
- 计算subscriptMode不为0（即上标或下标）的情况，算法：glyph.v0和glyph.v1乘以sizeShrinkage常量，然后根据上下标情况，上下偏移y坐标
- 若x + w > regionWidth，则将字符换行
- 若字符为空格：若BBCode对应是下划线，则替换为“下划线”；若对应中划线，则替换为“减号”；都不对应，则continue
- 处理纹理坐标
- 根据glyph.channel以不同方式计算顶点颜色
- 处理粗体和斜体的情况
- 处理下划线或中划线的情况
- **该方法最终会为每个字符，在uvs、cols、verts添加4个顶点数据，顺序为左下、左上、右上、右下**


    public class GlyphInfo
    {
        public Vector2 v0;          // 字形在生成的text mesh中左下角屏幕坐标
        public Vector2 v1;          // 字形在生成的text mesh中的右上角屏幕坐标
        public Vector2 u0;          // UV左下角坐标
        public Vector2 u1;          // UV右上角坐标
        public float advance = 0f;  // 从本字符到下个字符的步进宽度
        public int channel = 0;     // RGBA通道值，通常为15（1+2+4+8）
    }


注意在输出顶点数据前，已经调用`UpdateNGUIText`确保NGUIText中已保存所需的数据。

# 3. 一些要注意的点

- 使用动态字体时，Unity会生成要用到的字符的纹理，可能一开始是128×128的FontTexture就够了。若后面纹理不够用会重新生成一张新的更大的，触发[textureRebuildCallback](https://docs.unity3d.com/460/Documentation/ScriptReference/Font-textureRebuildCallback.html)，对应到`UILabel.OnFontTextureChanged`。这个方法会将所有引用到的label的字符，通过font.[RequestCharactersInTexture](https://docs.unity3d.com/460/Documentation/ScriptReference/Font.RequestCharactersInTexture.html)这个API去把所有字符都推到新的字体纹理中。接着再把所有label从panel移除再添加回去。【字体破碎现象很可能就是因为字体纹理重建引起的）
