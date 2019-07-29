title: NGUI源码学习——UIScrollView
date: 2017-05-01 21:42:55
categories:
- 技术
- 游戏开发
- Unity
- NGUI
tags:
- NGUI
- Unity
---
UIScrollView是NGUI支持滚动和拖拽的内容容器，可以和scroll bar联动。本文主要记录其相对滚动原理和核心方法，以及如何利用UIWrapContent优化滚动。了解了UIWrapContent的实现原理之后，可以在其上扩展和封装游戏UI中常见的数据展示控件。

【参考版本：NGUI 3.6.9】

<!-- more -->

# 1. 基础要点

UIScrollView依赖于UIPanel，必须和UIPanel绑在同一个go上。注意scrollView的内容物需要挂UIDragScrollView脚本，该脚本主要是接收OnPress和OnDrag事件，并转到UIScrollView中处理。

**相对滚动原理（`MoveRelative`为核心方法，所有的内容滚动最终都会调用到这里）：scrollView自身transform位置加偏移，同时panel的clipOffset减去相等的偏移量**（见下方代码）。这是因为一正一负的抵消可以让panel的位置保持不动，但由于panel offset的变化导致裁剪之后看上去物体好像被移动了一样。Panel的clipOffset属性使得在滚动的时候不用重建geometry，极大地提升滚动的效率。


    public virtual void MoveRelative (Vector3 relative)
    {
        mTrans.localPosition += relative;
        Vector2 co = mPanel.clipOffset;
        co.x -= relative.x;
        co.y -= relative.y;
        mPanel.clipOffset = co;

        // Update the scroll bars
        UpdateScrollbars(false);
    }


## 1.1 核心方法

- `LateUpdate`：更新scrollBar（如果有）的一些透明度数值；**若没有press**（即执行一些惯性动画的过程），则计算动量mMomentum（用于缓动）并插值出mScroll和offset值，利用`MoveAbsolute(offset)`来逐帧执行spring动画。最后根据restrictWithinPanel属性检测内容有没有超出边界，若有则调用`RestrictWithinBounds`
- `Press`：由挂了UIDragScrollView的go接收`OnPress`事件后通知UIScrollView，主要用于设置一些状态。按下时，mMomentum和mScroll清零，禁用Spring脚本，截掉tranform位置和panel offset的小数点（保持pixel-perfect），创建一个平面供drag投射用；松开时，限制bounds并触发一些回调。
- `Drag`：根据当前按下的点对上面的平面做投射，若有交点则计算拖拽产生的偏移，并使用`MoveAbsolute`和`RestrictWithinBounds`【见下方】方法来移动内容。

Awake时做一些属性自动调整。例如若panel的clipping为None会改为ConstrainButDontClip（即必须给panel限定一个范围）；自动调整Movement类型等等。

该类除了上面这些方法之外，主要就剩下一些和Scrollbar联动相关的代码，此处不赘述。

## 1.2 工具方法

- `RestrictWithinBounds`：将内容限制到scrollView的边界内。该方法会根据所有widget的内容包围盒以及panel的finalClipRegion，调用`NGUIMath.ConstrainRect(minRect, maxRect, minArea, maxArea)`计算出将内容rect限制到视口area所需的偏移。
    - 若用了`DragEffect.MomentumAndSpring`，则调用`SpringPanel.Begin`设置要移动的目标位置并启用动画
    - 否则直接MoveRelative移动内容

# 2. 使用UIWrapContent优化滚动效率

NGUI 3.7.x以上版本，有个新组件UIWrapContent，当列表内容很多时（甚至内容有无限多，或者循环滚动），可以用它来优化。用法很简单，和UIGrid或UITable等挂在同一层级下，包裹住内容即可。

注册`panel.onClipMove`事件（clipOffset改变时，基本上就是在滚动时触发）为`WrapContent`这个核心方法，其主要完成以下工作：

1. 根据UIScrollView的方向分为水平和垂直两种处理方式，两者原理一模一样，下面以水平为例。
2. 获取panel的本地corner坐标，令min=左下角-itemSize，max=右上角+itemSize，遍历每个孩子t进行如下处理：


    // max和min是panel尺寸加减一个物体尺寸的上下限
    float min = corners[0].x - itemSize;
    float max = corners[2].x + itemSize;
    Vector3 center = Vector3.Lerp(corners[0], corners[2], 0.5f);
    Transform t = mChildren[i];
    float distance = t.localPosition.x - center.x;

    // extents为所有内容bounds的半长，ext2即为尺寸
    if (distance < -extents)
    {
        Vector3 pos = t.localPosition;
        // 加上尺寸长度，移动到另一端【我们将这种操作称为item的跳转】
        pos.x += ext2;
        distance = pos.x - center.x;
        int realIndex = Mathf.RoundToInt(pos.x / itemSize);

        // 在设置面板中，min和max决定了滚动的上下限（可以为负，代表向左滚动的下限），两者相等则无限滚动
        if (minIndex == maxIndex || (minIndex <= realIndex && realIndex <= maxIndex))
        {
            t.localPosition = pos;
            UpdateItem(t, i);
            t.name = realIndex.ToString();
        }
        else allWithinRange = false;
    }
    /* 省略向右的情况...... */
    // 该选项决定是否剔除物体（实际上就是将超出范围的物体隐藏，将范围内的物体显示）以提高性能
    if (cullContent)
    {
        distance += mPanel.clipOffset.x - mTrans.localPosition.x;
        if (!UICamera.IsPressed(t.gameObject))
            NGUITools.SetActive(t.gameObject, (distance > min && distance < max), false);
    }

    protected virtual void UpdateItem (Transform item, int index)
    {
        if (onInitializeItem != null)
        {
            int realIndex = (mScroll.movement == UIScrollView.Movement.Vertical) ?
                Mathf.RoundToInt(item.localPosition.y / itemSize) :
                Mathf.RoundToInt(item.localPosition.x / itemSize);
            // 调用回调，其中index为物体在孩子列表中的索引，realIndex是物体最终位置所在的索引
            onInitializeItem(item.gameObject, index, realIndex);
        }
    }


下图是以1-9个数字方格为例的UIWrapContent滚动原理

![UIWrapContent向左滚动原理](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/Unity/UIWrapContent%E5%90%91%E5%B7%A6%E6%BB%9A%E5%8A%A8%E5%8E%9F%E7%90%86.jpg)

当滑动列表，每次有item跳转的时候，就会调用`onInitializeItem`回调（绑定你自定义的设置数据方法），此时就可以根据realIndex从你的数据列表中取得对应的数据，再将数据设置到go上（例如获取go的UILabel并设置其text等等）。

## 2.1 需要注意的点

- 如果列表是vertical的，由于坐标轴是上正下负，计算出来的realIndex从上到下是对应0到负数。**所以纵向列表取数据时，要将realIndex取绝对值再取数据。**
- 以vertical为例，要确保wrapContent的孩子个数为`panelHeight / itemHeight + 1`个，滚动设置自定义数据的行为才是正确的
- 参考设置：横向滚动时，`minIndex = 0, maxIndex = m_Data.Count`；纵向滚动时，`minIndex = 1 - m_Data.Count, maxIndex = 0`
