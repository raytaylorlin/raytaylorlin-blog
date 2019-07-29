title: NGUI源码学习——UISprite
date: 2017-06-17 22:33:44
categories:
- 技术
- 游戏开发
- Unity
- NGUI
tags:
- NGUI
- Unity
---
UISprite是NGUI最基本的图片显示控件之一。它的基类UIBasicSprite承载了绝大多数的显示逻辑，本文记录其各种填充方法的原理。要显示一个sprite，必定会有一个管理sprite的图集，最后会记录UIAtlas的一些基本要点。

【参考版本：NGUI 3.6.9】

<!-- more -->

# 1. 基类UIBasicSprite : UIWidget

核心是提供了一个总控Fill和一系列类型Fill方法（分为Simple、Sliced、Filled、Tiled、Advanced），供子类调用。Fill方法接收geometry的顶点、UV和颜色列表并填充相应数据。该方法会在UIWidget.UpdateGemoetry中调用OnFill时调用。

- flip属性：有水平、垂直、两者皆翻转，通过交换uv坐标的位置来实现
- `SimpleFill`：获取drawingDimensions，drawingUVs，drawingColor属性，直接填充4个顶点、uv和颜色
- `SlicedFill`：九宫格填充，所以会绘制9个区域。核心填充方法见下图和代码。其中`mTempPos`为图中所标4个点，通过左下角、右上角点和border计算所得；`mTempUVs`为图中所标4个点的UV，通过outer和inner矩形及一个简单的工具方法`NGUIMath.ConvertToTexCoords`计算所得。

![SlicedFill九宫格填充原理](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/Unity/SlicedFill%E4%B9%9D%E5%AE%AB%E6%A0%BC%E5%A1%AB%E5%85%85%E5%8E%9F%E7%90%86.jpg)

    for (int x = 0; x < 3; ++x)
    {
        int x2 = x + 1;
        for (int y = 0; y < 3; ++y)
        {
            if (centerType == AdvancedType.Invisible && x == 1 && y == 1) continue;
            int y2 = y + 1;
            // 分别填充顶点，UV，颜色
            verts.Add(new Vector3(mTempPos[x].x, mTempPos[y].y));
            verts.Add(new Vector3(mTempPos[x].x, mTempPos[y2].y));
            verts.Add(new Vector3(mTempPos[x2].x, mTempPos[y2].y));
            verts.Add(new Vector3(mTempPos[x2].x, mTempPos[y].y));
            uvs.Add(new Vector2(mTempUVs[x].x, mTempUVs[y].y));
            uvs.Add(new Vector2(mTempUVs[x].x, mTempUVs[y2].y));
            uvs.Add(new Vector2(mTempUVs[x2].x, mTempUVs[y2].y));
            uvs.Add(new Vector2(mTempUVs[x2].x, mTempUVs[y].y));
            cols.Add(c);
            cols.Add(c);
            cols.Add(c);
            cols.Add(c);
        }
    }


- `TiledFill`：平铺填充。通过两层while循环，使用inner矩形（即九宫格的中间区域）一行一行地平铺填充
- `FilledFill`：主要由mfillDirection（填充方式）、mfillAmount（填充量，范围0-1代表0-90°）和mInvert（是否反转）来控制填充的效果，见下图示例。其中的计算涉及各种三角函数计算（`RadialCut`方法），不赘述。Radial 180需要分左右画2个矩形，而Radial 360则需要分四块画4个矩形。显然，Radial 360可以实现类似技能CD的效果。

![各种FilledFill类型对比](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/Unity/%E5%90%84%E7%A7%8DFilledFill%E7%B1%BB%E5%9E%8B%E5%AF%B9%E6%AF%94.jpg)

- `AdvancedFill`：高级填充，可以对上下左右中分别单独设置

而UISprite除了drawingDimesions这个属性有一些将宽高处理为偶数（pixel-perfect）的逻辑之外，本身没有什么特别的功能，绝大多数功能都在UIBasicSprite中。

# 2. UIAtlas

每个sprite都是显示图集atlas中的一块，因此总会持有mAtlas的引用以便用mSpriteName快速获取UISpriteData数据。`UISpriteData`类是个简单的数据结构，包含名字、图集所在坐标和尺寸、border（九宫格相关）、padding。

该类维护了一个`List<UISpriteData> mSprites`和`Dictionary<string, int> mSpriteIndices`（sprite名到index的映射）。`GetSprite`方法会优先在字典中查索引，否则在mSprites列表中查。途中如果字典和列表不一致或查找不到时，调用`MarkSpriteListAsChanged`，用mSprites去重建字典的索引。
