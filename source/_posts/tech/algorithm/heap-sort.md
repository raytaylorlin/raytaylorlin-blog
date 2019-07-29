title: 算法学习笔记：堆排序
date: 2014-07-10 11:11:05
categories:
- 技术
- 算法
tags:
- 算法
- 堆排序
---
相比归并排序和快速排序，本文将介绍另一种平均时间复杂度是$O(n \log n)$的排序方法——堆排序（Heap Sort）。堆排序使用了一种被称为“堆”的数据结构，这也是它相比其他两种排序方法的特殊之处。堆这种数据结构不仅可以用于排序，也可以用来维护优先级队列。本文最后还简要对比了快速排序和堆排序的优缺点。

<!-- more -->

# 1. 堆

## 1.1 堆的性质

堆是一棵**完全二叉树**，实际中可以通过一个数组来实现，它最重要的一个性质是：**任意节点都小于（大于）等于其子节点**。将根节点最小的堆称为最小堆，根节点最大的堆称为最大堆。下图给出了一个最大堆的示例及其数组表示，可以直观地看出每个节点都比它的孩子们都要大。

![最大堆示例及其数组表示](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/algorithm/%E6%9C%80%E5%A4%A7%E5%A0%86%E7%A4%BA%E4%BE%8B%E5%8F%8A%E5%85%B6%E6%95%B0%E7%BB%84%E8%A1%A8%E7%A4%BA.jpg)

在上图中可以看到，完全二叉树的节点可以从根节点编号为1开始按顺序排列，对应数组A中的索引（注意此处下标是从1开始的）。给定一个节点`i`，我们很容易可以得到它的左孩子是`2i`，右孩子是`2i+1`，父节点是`i/2`

## 1.2 堆的基本操作

堆有两种基本操作（下面以最小堆为例）：

* 插入元素k：直接将k添加到数组最后，然后向上冒泡（bubble-up）调整堆。**向上冒泡操作：将要调整的元素与其父节点比较，如果大于其父节点则交换，直到恢复堆的性质**
* 提取最值：最值即根元素。然后将其删除，令根元素=最后的叶子结点元素，然后从根元素开始向下冒泡（bubble-down）调整堆。**向下冒泡操作：每次应该从要调整节点，其左右孩子一共三个节点中选择最小的子节点来交换（如果最小就是其本身就不用交换），直到恢复堆的性质**

实际中经常需要将一个包含n个元素无序数组建立成堆，下面的Heap类中的构造方法将展示如何通过`_bubbleDown`向下冒泡调整来建堆。堆实质上是一棵完全二叉树，树高总为$\log n$，每种基本操作的耗时操作都在于冒泡调整以满足堆的性质，因此它们的时间复杂度都是$O(n \log n)$。

    class Heap():
        def __init__(self, heap=[]):
            self._heap = heap
            # 从堆的一半开始逐个节点向前调整
            # （因为完全二叉树的后一半节点都是叶子节点，不需要调整）
            for i in reversed(xrange(len(self._heap) / 2)):
                self._bubbleDown(i)

        def insert(self, data):
            """在堆中插入一个元素"""
            self._heap.append(data)
            self._bubbleUp(len(self._heap) - 1)

        def extract(self):
            """提取堆的最值"""
            # 根元素总是堆的最值
            result = self._heap[0]
            if self.size() > 1:
                # 交换根元素和最后一个元素，并删除掉最后一个元素
                self._heap[0] = self._heap.pop()
                # 交换之后可能会破坏堆的性质，需要向下调整根元素
                self._bubbleDown(0)
            return result

        def _bubbleUp(self, i):
            """对指定下标的元素向上进行调整，以维护堆的性质"""
            parent = (i - 1) / 2
            # 冒泡终止条件：到达根节点或者已满足堆的性质
            while parent >= 0:
                if self._heap[i] < self._heap[parent]:
                    self._heap[i], self._heap[parent] = self._heap[parent], self._heap[i]
                    i = parent
                    parent = (i - 1) / 2
                else:
                    break

        def _bubbleDown(self, i):
            """对指定下标的元素向下进行调整，以维护堆的性质"""
            smallest = i
            # 注意此处左右孩子的算法和教科书不太一样，因为实际数组下标是从0开始
            left, right = 2 * i + 1, 2 * i + 2
            # 找出i，left，right三个元素最小的一个
            if left < self.size() and self._heap[left] < self._heap[smallest]:
                smallest = left
            if right < self.size() and self._heap[right] < self._heap[smallest]:
                smallest = right
            if smallest != i:
                # 交换位置，并递归地继续向下冒泡调整
                self._heap[i], self._heap[smallest] = self._heap[smallest], self._heap[i]
                self._bubbleDown(smallest)

## 1.3 堆排序

有了上面定义的Heap类之后，堆排序将变得非常简单。堆的性质决定了其根元素总是最小值，而Heap类的`extract`方法总是完成提取最小值并调整堆的操作，因此使用堆来排序只需要先建堆，再执行n次`extract`操作即可，堆排序的时间复杂度是$O(n \log n)$。

    def heapSort(data):
        heap = Heap(data)
        return [heap.extract() for i in xrange(len(data))]

实际上，堆排序的原理非常像$O(n^2)$的选择排序——两者每一轮都选择最小的一个数，只不过堆排序选择这个数（并调整堆）只要花费$O(\log n)$，而选择排序选择这个数要遍历所有的数花费$O(n)$。

# 2. 堆的应用：优先队列

优先队列（priority queue）实际上也是由堆来实现，所以有最大优先队列和最小优先队列两种。优先队列的其中一个应用就是在共享计算机系统的作业调度。最大优先队列记录将要指定的各个作业及它们之间的相对优先级。当一个作业完成或者被中断后，调度器调用`extract`从所有的等待作业中，选出具有最高优先级的作业来执行。在任何时候，调度器可以调用`insert`把一个新作业加入到队列中来。

最小优先队列可以诶用于基于事件驱动的模拟器，队列中保存要模拟的事件，每个事件都有一个发生时间作为其*关键字*。事件必须按照发生的时间顺序模拟，而某一事件的模拟结果可能会触发其他事件。模拟程序调用`extract`来选择下一个要模拟的事件，当一个新事件产生时，模拟程序调用`insert`将其插入最小优先队列中。

从上面的应用可以看出，**堆最好用的地方在于其提取最值的速度非常快。因此当算法中需要不断重复计算获取最小（大）值时，就可以考虑使用堆这种数据结构。**

# 3. 堆排序和快速排序的比较

尽管堆排序和快速排序都是$O(n \log n)$的排序算法，但这只是算法渐近的趋势，实际上它们的性能（主要表现在常数因子上）还是有一些区别的。下表给出了快速排序、堆排序和插入排序比较次数和交换次数的对比：（数据来自[Comparing Quick and Heap Sorts](https://www.cs.auckland.ac.nz/~jmor159/PLDS210/qsort3.html)）

<table class="table table-bordered responsive" border="1">
  <tbody><tr><th rowspan="2"><i>数据规模</i></th>
   <th colspan="2">快速排序</th>
   <th colspan="2">堆排序</th>
   <th colspan="2">插入排序</th></tr>
   <tr><th>比较次数</th><th>交换次数</th>
    <th>比较次数</th><th>交换次数</th>
    <th>比较次数</th><th>交换次数</th></tr>
    <tr><td>100</td><td>712</td><td>148</td><td>2,842</td><td>581</td><td>2,595</td><td>899</td></tr>
    <tr><td>200</td><td>1,682</td><td>328</td><td>9,736</td><td>1,366</td><td>10,307</td><td>3,503</td></tr>
    <tr><td>500</td><td>5,102</td><td>919</td><td>53,113</td><td>4,042</td><td>62,746</td><td>21,083</td></tr>
</tbody></table>

可见快速排序的比较次数和交换次数都要比堆排序少，性能更优，这也是为什么多数商业应用选择快速排序作为排序算法的原因。然而，当实际应用中**需要保证一定的响应时间**时，就不应该使用快速排序，因为它最坏情况下是一个$O(n^2)$的算法（要记住在某些应用中最坏情况是会经常出现的，这时候你就不能依赖快速排序的平均性能）。如果数据规模n比较少，插入排序是更好的选择，因为它的代码非常简单，而且常数因子也更小。如果n很大，毫无疑问应该选择堆排序，它保证了$O(n \log n)$的时间复杂度。在一些像医疗监控、航天航空、工业生产等有关键任务的行业系统中，必须总是考虑最坏情况的出现，而不是平均情况。

参考文献：机械工业出版社《算法导论（第3版）》第6章  

