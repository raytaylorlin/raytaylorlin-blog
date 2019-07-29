title: 算法学习笔记：快速排序
date: 2014-07-05 11:11:05
categories:
- 技术
- 算法
tags:
- 算法
- 快速排序
---
通常算法初学者都会接触到形形色色的排序算法，例如插入排序、选择排序、冒泡排序等等，这些排序算法都非常容易理解和上手，但是它们的时间复杂度是$O(n^2)$的。本文将介绍快速排序，同归并排序一样，它的时间复杂度是$O(n \log n)$。

对于包含n个数的输入数组来说，快速排序是一种最坏情况运行时间为$O(n^2)$的算法，但**它通常是实际应用中最常用的排序算法**，因为它的平均性能是$O(n \log n)$的，而且隐含的常数因子非常小，并能够进行原址排序。

<!-- more -->

# 1. 快速排序思想

与归并排序一样，快速排序也使用了分治法的思想，依然遵循分治法三部曲：

* 分解：选定数组A中一个元素A[m]，A[l...r]被划分为两个（可能为空）子数组，其中数组A[l...m-1]所有元素都小于等于A[m]，数组A[m+1...r]所有元素都大于等于A[m]。计算下标m也是分解过程的一部分。
* 解决：对两个子数组递归A[l...m-1]和A[m+1...r]调用快速排序
* 合并：子数组都是原址排序，不需要合并操作，A[l...r]本身已有序

算法的关键在于选取主元（pivot element），即上面提到的划分数组的元素。例如对[3,8,2,5,1,4,7,6]进行第一趟快速排序，
选择第一个元素3作为主元，左边都是小于3的元素，右边都是大于3的元素，即[2,1,3,6,7,4,5,8]，接着再对左右两部分（[2,1]和[6,7,4,5,8]）递归调用快速排序。快速排序核心算法如下：

    def quickSort(data, left, right):
        # 递归中止条件：数组只有1个元素
        if left < right:
            pivotLastIndex = partition(data, left, right)
            quickSort(data, left, pivotLastIndex - 1)
            quickSort(data, pivotLastIndex + 1, right)

# 2. 选取主元和重排

其中算法的关键部分是partition方法，该方法选取一个主元，并围绕着该主元实现了对子数组data[left...right]的重排。partition方法如下：

    def partition(data, left, right):
        # 选取主元
        pivotIndex = chooseFirstPivotIndex(left, right)
        pivot = data[pivotIndex]
        # 预处理：将主元放到数组第一个位置
        data[left], data[pivotIndex] = data[pivotIndex], data[left]

        # 从左到右将未调整部分的元素放到已调整的部分中去
        i, j = left + 1, left + 1
        while j <= right:
            # 发现比主元小的元素，则将其交换到前面的位置
            if data[j] < pivot:
                data[i], data[j] = data[j], data[i]
                i += 1
            j += 1
        # 记住主元数一直都位于左边界，最后要把它放到合适的中间位置
        pivotLastIndex = i - 1
        data[left], data[pivotLastIndex] = data[pivotLastIndex], data[left]
        return pivotLastIndex

    def chooseFirstPivotIndex(left, right):
        return left

为了方便起见，上述partition方法中，一直都选取第一个元素作为主元（`chooseFirstPivotIndex`方法），后面会看到总是选取第一个元素作为主元并不是最优的选择。**如果不是选择第一个元素作为主元，我们都将主元换到第一个位置方便处理。**partition方法运用了一种巧妙的方式（使用i,j两个指针）确保只要用$O(n)$的时间就能重排data[left...right]，使得data呈现[小于pivot元素，pivot，大于pivot元素]的排列方式。

将数组看做`[p, ...<p..., ...>p..., ...rest...]`，p为主元，`<p`和`>p`部分为已调整元素，`rest`部分为未调整元素。整个重排过程就是从左到右将未调整部分的元素放到已调整的部分中去。令i和j都为1：  
当A[j]>p时：j++  
当A[j]&lt;p时：交换A[j]和A[i]，i++，j++  

通过下面的图例可以比较清晰地看到partition具体的重排过程。

![快速排序一趟重排示例](http://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/algorithm/%E5%BF%AB%E9%80%9F%E6%8E%92%E5%BA%8F%E4%B8%80%E8%B6%9F%E9%87%8D%E6%8E%92%E7%A4%BA%E4%BE%8B.jpg)

# 3. 性能分析

当数组接近逆序时，选取第一个元素作为主元的方案会使快排的时间上升到$O(n^2)$，因为逆序的数组第一个元素会重排剩余的所有元素，而剩余的元素又全都是逆序的。为了解决这个问题，可以每次都随机选取数组内的一个元素作为主元。在上面的代码中，只需要将`chooseFirstPivotIndex`方法换成下面的`chooseRandomPivotIndex`方法即可：

    def chooseRandomPivotIndex(left, right):
        import random
        return random.choice(range(left, right + 1))

证明：随机化快速排序算法的运行时间为$O(n \log n)$。

1、 令$C(\sigma )$=选取的主元为$\sigma $时快排的比较次数，${z\_i}$=数组中第i小的元素，$X_{ij}(\sigma )$=选取的主元为$\sigma $时，${z_i}$和${z_j}$比较的次数。**注意：数组中任意两个元素之间比较最多只有1次（因为当其中1个元素被选为主元时，必定会比较一次，如果2个元素都不是主元，它们不会被比较到）**，则有：

$$\forall \sigma ,C(\sigma ) = \sum\limits\_{i = 1}^{n - 1} {\sum\limits\_{j = i + 1}^n { {X\_{ij}}(\sigma )} } $$

$$E[C] = \sum\limits\_{i = 1}^{n - 1} {\sum\limits\_{j = i + 1}^n {E[{X\_{ij}}] = } } \sum\limits\_{i = 1}^{n - 1} {\sum\limits\_{j = i + 1}^n { {P\_r}[{X\_{ij}} = 1]} } $$

2、只要计算第i小和第j小元素之间比较1次的概率。  
关键：$\forall i < j,{P\_r}[{X\_{ij}} = 1] = {2 \over {j - i + 1}}$  
原因：考虑${z\_i},{z\_{i + 1}},...,{z\_{j - 1}},{z\_j}$（一共j-i+1个元素），
当选中的主元是${z\_i}$或${z\_j}$，那么${z\_i}$和${z\_j}$必定会比较到；当选中的主元是其它，那么${z\_i}$和${z\_j}$不会比较到，所以概率为${2 \over {j - i + 1}}$，所以

$$E[C] = 2\sum\limits\_{i = 1}^{n - 1} {\sum\limits\_{j = i + 1}^n { {1 \over {j - i + 1}}} } $$

3、对于每个固定的i，有$\sum\limits\_{j = i + 1}^n { {1 \over {j - i + 1}}}  = {1 \over 2} + {1 \over 3} + ... + {1 \over n} = \sum\limits\_{k = 2}^n { {1 \over k}}  \le \ln n$

（关于$\sum\limits_{k = 2}^n { {1 \over k}}  \le \ln n$见下图解释）

![分数求和不等式图解](http://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/algorithm/%E5%88%86%E6%95%B0%E6%B1%82%E5%92%8C%E4%B8%8D%E7%AD%89%E5%BC%8F%E5%9B%BE%E8%A7%A3.jpg)

$$\sum\limits\_{k = 2}^n { {1 \over k}}  \le \int\_1^n { {1 \over x}dx = \ln x|\_1^n = \ln n} $$

综上所述，$E[C] \le 2 \cdot n \cdot \ln n = O(n\log n)$

QED


参考文献：机械工业出版社《算法导论（第3版）》第7章
