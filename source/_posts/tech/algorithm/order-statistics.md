title: 算法学习笔记：顺序统计量
date: 2014-07-14 10:24:05
categories:
- 技术
- 算法
tags:
- 算法
- 顺序统计量
---
在一个由n个元素组成的集合中，第i个*顺序统计量（order statistics）*是该集合中第i小的元素，例如第1个顺序统计量是最小值，第n/2个顺序统计量是中位数（当n是偶数时，中位数一般指i=n/2的那个数）。顺序统计量问题就是给定包含n个（互异的）数的集合和整数i，找出第i小的数。

解决这个问题最直接的办法就是先排序，再直接找出第i个元素即可，排序最快的时间复杂度是$O(n \log n)$。但其实还可以更快地找到第i个顺序统计量，本文将主要介绍一个实用的随机算法和确定性算法，它们都可以在$O(n)$的时间内解决这个问题。

<!-- more -->

# 1. 最小值和最大值

在一个有n个元素的集合中，需要做$n-1$次比较才能确定其最小（大）元素，这确实是最好的结果。但如果要同时找出最小值和最大值，通常的做法是对每一个输入的元素都与已知的min和max作比较，这样相当于独立完成两趟查找进行$2(n-1)$次比较。

事实上，只要记录已知的min和max，每次取2个元素进行比较，然后较小的和min进行比较，较大的和max进行比较，这样就实现了对每两个元素进行3次比较，总共只需要$3\left\lfloor {n/2} \right\rfloor $次比较。至于初始化min和max，如果n是奇数，可以初始化min和max都为集合的第一个数；如果n是偶数，则取前两个数进行比较，小的为min，大的为max。这种算法虽然也是渐近为$O(n)$的算法，但从比较次数来看要比暴力求解快上约25%。

# 2. 随机化选择算法

下面要介绍的选择第i小的算法，是基于[快速排序](/Tech/algorithm/quick-sort/)的。在进行主元划分之后，快速排序会递归处理划分的两边，而随机化选择算法只处理划分的其中一边。该算法的可以以$O(n)$的时间找到第i小的元素。

快速排序的划分主元方法`partition`会选取一个主元pivot，并围绕着该主元实现了对子数组data[left...right]的重排。假设最终pivot会处于**下标`pivotLastIndex`处（绝对位置，相对下标0而言）**，以及**下标`pivotRelativeIndex`处（相对位置，相对下标left而言）**，则重排之后data[left...pivotLastIndex-1]的元素都会比pivot小，data[pivotLastIndex+1...right]的元素都会比pivot大。而我们要找第i小的元素，最理想的情况就是i等于pivotRelativeIndex刚好是要找的元素；要么i落在左部分，则递归地查找左部分第i小的元素；要么i落在右部分，则递归地查找右部分第i-pivotRelativeIndex小的元素。下面的代码展示了具体的随机化选择算法，因为数组下标都是从0开始的，所以要特别注意**我们口头所说的第i小元素，在程序里面实际是第i-1小的元素，以及其他一些关于下标计算的细节**。

    def randomizedSelect(data, left, right, i):
        # 递归中止条件：数组只有1个元素
        if right == left:
            return data[left]

        # pivotLastIndex是主元在data中的索引
        pivotLastIndex = partition(data, left, right)
        # pivotRelativeIndex是主元在data中相对于当前left的索引
        pivotRelativeIndex = pivotLastIndex - left

        if i == pivotRelativeIndex:
            # 主元的绝对位置刚好是要查找的第i小元素
            return data[pivotLastIndex]
        elif i < pivotRelativeIndex:
            # 第i小的元素位于左半部分，递归查找左半部分的第i小的元素
            return _randomizedSelect(data, left, pivotLastIndex - 1, i)
        else:
            # 第i小的元素位于右半部分，递归查找右半部分的第i-pivotRelatvieIndex小的元素
            return _randomizedSelect(data, pivotLastIndex + 1, right, i - pivotRelativeIndex - 1)

运行时间分析：最好的情况就是每次选到的pivot都是数组的中位数。$T(n) \le T({n \over 2}) + O(n)$，根据主方法第2种情况，运行时间为$O(n)$。随机化选取基准，和随机化快排一样，平均运行时间也是$O(n)$。最坏情况下，每次划分都极不走运地总是按余下元素中最大的来进行划分，这样就会和快速排序一样退化到$O(n^2)$。

# 3. 确定性选择算法

从上面的分析可以看出`randomizedSelect`算法可能出现最坏情况的原因是`partition`有可能划分得不够理想。而确定性选择算法`deterministicSelect`则对`partition`算法进行修改，使用*中位数的中位数*作为pivot，使得每次划分都尽量让pivot把数组划分成1:1的状态。

在快速排序中，定义了`partition`方法大致如下：

    def partition(data, left, right):
        # 选取主元
        pivotIndex = chooseRandomPivotIndex(left, right)
        # 后面省略...

确定性选择算法是把`chooseRandomPivotIndex`换成可以选择中位数的中位数的`chooseMedianPivotIndex`，算法流程如下：

1. 将n个元素划分为m=n/5组，每组5个元素（最多有一组由剩下的n mod 5个元素组成）
2. 对m组元素都进行插入排序（每组只有5个数所以很快），再找出各组的中位数，共m个
3. m个中位数组成临时数组C，递归调用deterministicSelect找出C的中位数
4. 中位数的索引作为chooseMedianPivotIndex的返回结果

确定性选择算法最坏情况也可以达到$O(n)$，**但实际并没有随机算法快，因为它忽略的常数项更大，而且要多耗费存储空间。**

参考文献：机械工业出版社《算法导论（第3版）》第9章  

