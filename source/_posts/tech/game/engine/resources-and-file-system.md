title: 游戏引擎中的资源与文件系统
date: 2016-06-17 14:10:20
categories:
- 技术
- 游戏开发
- 游戏引擎
tags:
- 游戏引擎
- 架构
---
载入及管理多种媒体，是游戏引擎必须具备的能力。多数引擎会采用某种类型的资源（或资产）管理器，载入并管理游戏所需的资源，并确保在同一时间每个媒体文件只可载入一份。每个资源管理器都会大量使用文件系统。本文将介绍现代三维游戏引擎中的各种文件系统API，再分析典型资源管理器的运作方式。

<!-- more -->

# 1. 文件系统

## 1.1 文件名和路径

关于文件和文件夹路径的概念，绝对路径和相对路径的概念，它们在各种操作系统之间的区别，属于常识范畴，此处不赘述。

关于搜寻路径，是指含若干个路径（以特殊字符分隔）的字符串，寻找文件时会从这些路径逐个寻找，PATH环境变量就是一种搜寻路径。在运行期搜寻资产是费时的做法，而通常资产路径会在运行期之前就得知，所以应该完全避免搜寻资产。

关于路径API，一般用于对路径进行多种操作，如分离“目录/文件名/扩展名”、使路径规范化、绝对和相对路径互转等等。游戏引擎通常会实现或封装轻量化的路径处理API，以便实现跨平台，从各种特殊的储存媒体（如记忆棒、DVD盘、网络文件系统等等）中存取数据，以及提供操作系统API未能提供的功能，如串流（即在游戏运行中同时载入数据）。

## 1.2 基本文件I/O

### 1.2.1 文件I/O API

许多游戏引擎都会把文件I/O API封装成自定义的API，这样至少有三个好处：保证I/O API在所有目标平台上均有相同行为；API可以简化到只剩下实际需要的函数，使维护开支维持最小限度；可提供延伸功能，如处理各种特殊的储存媒体（同自定义路径处理API）。

每次调用输入/输出，都需要称为缓冲区的数据区块，以供程序和磁盘之间传送字节。当API负责管理数据缓冲，就称之为有缓冲功能的API，否则为无缓冲。C标准程序库中，以f开头的文件API是带缓冲的，如`fopen()`，没有f开头是无缓冲的，如`read()`。有时自行管理缓冲区是有必要的。例如往日志写数据可能会显著降低性能，可以先把数据累积在内存缓冲，满溢后才写进盘内，甚至把缓冲输出函数置于另一线程里，以避免令主游戏循环发生流水线停顿。

### 1.2.2 同步与异步

C标准库的两种文件I/O库都是同步的，即程序发出I/O请求以后，必须等待读/写数据完毕，程序才能继续运行。为了提高用户体验，往往要使用串流来载入资源，这必须使用异步文件I/O库。多数异步I/O库容许主程序在请求发出后一段时间，等待I/O操作完成才继续运行。有些异步I/O库容许程序员取得某异步操作所需时间的估算，一些API也可以为请求设置时限，并设置请求超时的安排（例如取消请求、通知程序、继续尝试等）。

异步I/O操作常有不同的优先权，例如从硬盘中串流音频，并且在串流其他资源时播放音频，显然前者优先权高于后者。异步I/O系统必须能暂停较低优先权的请求，才可以让较高优先权的I/O请求有机会在时限前完成。

关于异步操作（不局限于文件I/O）的实现原理，一般是利用另一线程进行同步操作来实现。主线程调用异步函数时，会把请求放入一个队列，并立即传回。同时，I/O线程从队列中取出请求，并以阻塞I/O函数处理这些请求。请求的工作完成后，就会调用主线程之前提供的回调函数告之该操作己完成。若主线程选择等待完成I/O请求，就会使用信号量处理（每个请求对应一个信号量，主线程把自身处于休眠状态，等待I/O线程在完成请求工作后通知信号量）。

# 2. 资源管理器

资源管理器由两部分组成：一部分负责管理离线工具链，用来创建资产并把它们转换成引擎可用的形式；另一部分在执行期管理资源，确保资源在使用前已载入内存，不需要时从内存卸下。

## 2.1 离线资源管理与工具链

### 2.1.1 资产的版本控制

有些游戏团队使用源码版本控制工具来管理资源。艺术资产通常有极大的数据量，直接从中央版本库复制到本地往往是低效的。以下是一些参考解决方案：

* 使用如Alienbrain这种特别针对极大量数据的商业VCS
* 在VCS上精心设计一套系统，保证用户只会取得其真正所需的文件到本地
* 顽皮狗开发了一款私有工具。用户拥有资产版本库的完整本地视图，只要文件未签出，本地就一直是UNIX的符号链接（Windows可以使用junction实现）以消除数据复制。当签出文件时则移除符号链接，更换为本地副本，签入时则相反。

### 2.1.2 资源数据库

游戏引擎不会使用多数资产原本的格式，而是需要通过一些资产调节管道转换资产，转换过程中每个资源都会产生**元数据**描述如何对资源进行处理。例如描述压缩纹理时，使用哪种压缩方法；描述导出动画片段时，导出哪个范围的帧。大型游戏需要“资源数据库”来管理资源管道所需的数据。无论采用什么形式，数据库都需要提供以下功能：

* 以一致的方式处理多种类型的资源
* 创建、删除、查看、移动磁盘位置和修改资源
* 资源交叉引用其他资源，并维持数据库内的引用完整性
* 保存版本历史，含完整日志记录、改动者及事由
* 支持不同形式的搜索和查询

### 2.1.3 一些成功的资源数据库设计

* 虚幻3：由万用工具UnrealEd管理，它是引擎的一部分
    * 优点：创建资产后能立即看到资产在游戏中运行的模样；以单一、整合、一致的界面管理所有类型的资源；资产必须明确导入数据库，制作初期便可检查资源有效性
    * 缺点：所有资源存于少量的大型二进制包文件，不利于VCS合并；资源重命名或移动时，使用虚拟对象，即把旧资源映射到新名称/位置，问题是虚拟对象会闲置、累积起来造成问题，尤其是删除资源时变得严重
* 顽皮狗的《神秘海域》引擎
    * 最初使用MySQL存储资源元数据，并编写自制GUI工具Builder管理，后改用Perforce以提供版本控制，元数据改为XML
    * Builder管理演员（包含行为的动态对象）和关卡（含静态背景网格和关卡信息等）两种类型的资源，动画可以组成名为动画包（buddle）的伪文件夹
    * 引擎含一组基于命令行的工具，用于查询数据库，处理资源原生DCC文件，生成某演员或关卡
    * 优点：资源粒度小；Builder仅提供必需的特性；源文件映射显而易见，用户容易得知某资源由哪些资产而来；容易更改DCC数据的到处及处理方式；依赖系统会自动处理，生成资产非常容易
    * 缺点：欠缺预览资产的可视化工具；各种类型的工具没有完全整合
* OGRE：拥有一个颇完备、设计非常好的运行时资源管理器，通过一组简单一致又有扩展性的接口就能载入任何类型的资源。缺点在于仅是运行时方案，本身提供的离线处理很弱
* 微软的XNA：通过VS IDE的项目管理及生成系统，把游戏资产以同样形式管理及生成

### 2.1.4 资产调节管道

资产调节管道用于将DCC原生格式文件转换成引擎可用的形式，一般经过3个处理阶段：

1. 导出器：为DCC工具编写自定义插件，将数据导出为某种中间格式。如果DCC不提供自定义方法，则应该把数据存成开放格式，或比较直观的文本格式，或其他可做反向工程的原生格式
2. 资源编译器：对DCC导出的数据进行一定处理，如把网格的三角形重新排列成三角形带，或压缩纹理。并非所有数据都要编译
3. 资源链接器：将多个资源先结合成单个有用的包，如复杂的三维模型，然后才载入至游戏引擎。并非所有数据都要链接

如同程序的源文件，各资产之间也有依赖关系，例如某网格引用若干个材质，这些材质又引用多个纹理。这些依赖关系通常会影响资产在管道内的处理次序，也可告诉我们，当某个源资产做出改动后，要重新生成哪些资产。每个资产调节管道都需要一组规则来描述资产间的依赖关系，并自己搭建系统或使用像make这样的工具来以正确顺序生成资产。一定要管理好资产间的依赖。

## 2.2 运行时资源管理

### 2.2.1 运行时资源管理器的责任

* 确保任何时候，同一个资源在内存中只有一份副本
* 管理每个资源的生命期
* 处理复合资源的载入（如三维模型）
* 维护引用完整性：包括单个资源内的交叉引用，以及资源间的交叉引用
* 管理资源载入后的内存用量，确保资源储存在内存中合适的地方
* 容许按资源类型，载入资源后执行自定义的处理
* 通常提供统一的易扩展的接口管理多种资源类型
* 若引擎支持，则要处理串流

### 2.2.1 资源文件及目录组织

资源一般储存为磁盘上的文件，并位于使创作者方便而组织的树状目录中。但引擎通常不会理会资源被放置于资源树中的哪个位置，引擎会把多个资源包裹为单一文件，这种手法能将寻道时间、开启每个文件的时间、从文件读至内存的时间都降到最低。

OGRE使用ZIP存档资源，因为ZIP是开放格式，内部虚拟文件有相对路径，可被压缩（载入数据后解压所花的时间，通常比读取无压缩数据所花的时间少），并可视为模块（例如把需要本地化的资产打包，针对不同语言制作不同版本的ZIP）。虚幻3采取类似的手法，但是其所有资源都必须置于大型的pak自定义格式文件中，并不容许资源以盘上独立文件出现。

### 2.2.2 资源文件格式和GUID

每类资源都可能有不同的文件格式。单一文件格式也可储存多种不同类型的资产，如Granny的文件格式可轻易用来储存任何种类的数据。许多引擎会自定义文件格式，因为引擎所需部分信息可能没有标准格式可以支持，以及对资源脱机处理，以让其遵从某种内存布局加速运行时载入。

所有资源都需要GUID来识别，最常见就是使用资源的文件系统路径（操作系统保证两个文件不能有相同的路径），也有使用128位散列GUID的。虚幻3的GUID格式是包名和包内资源路径串接而成，像《战争机器》的一个资源GUID为`Locust_Boomer.PhysicalMaterials.LocustBommerLeather`。

### 2.2.3 资源注册表

资源管理器都含某种形式的资源注册表，以**保证在任何时间，载入内存的每个资源只会有一份副本**。最简单的实现方法是使用字典，键为资源的GUID，而值是指向内存中资源的指针。资源载入内存时，加进资源注册表字典。卸下资源时，就删除其注册表记录。

若不能从表中找到请求的资源，最直觉的处理手法就是自动载入该资源。但这样做可能会因为临时从硬盘或光驱等缓慢设备读取数据而严重拖慢游戏帧率。因此引擎可采取这两种替代手法：游戏进行中完全禁止加载资源（游戏关卡的所有资源在游戏进行前全部加载，那时候通常是loading界面）；或资源以相对较难实现的异步形式加载，如玩关卡A时，关卡B的资源在后台加载。

### 2.2.4 资源生命期

资源管理器的职责之一是自动管理资源生命期，或对游戏提供所需API供手动管理。每个资源对生命期有不同需求：游戏持续的所有时间（如角色网格、纹理、动画，HUD的纹理字形等等），持续某一关卡的时间，短于所在关卡的时间（如过场动画），即时串流（如BGM、环境音效等）。

某资源的载入时期通常在玩家第一次看见该资源便能决定，但何时卸下资源归还内存，就难以回答，因为可能存在多个关卡共享的资源。解决方案之一就是对资源引用计数，即载入新关卡时，遍历所需资源并引用加1，再遍历即将结束的关卡的资源，所有引用减1。下图给出了资源引用计数的例子。

![载入或卸下两个关卡时资源的引用变化](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image%2Fengine%2F%E8%BD%BD%E5%85%A5%E6%88%96%E5%8D%B8%E4%B8%8B%E4%B8%A4%E4%B8%AA%E5%85%B3%E5%8D%A1%E6%97%B6%E8%B5%84%E6%BA%90%E7%9A%84%E5%BC%95%E7%94%A8%E5%8F%98%E5%8C%96.jpg)

### 2.2.5 资源的内存管理

资源加载的内存位置可能不同，像纹理、顶点缓冲、着色器驻留在显存，大部分资源驻留在主内存。设计游戏引擎时，有时用已有的内存分配器来设计资源系统，有时则要让内存分配器配合资源管理所需。

* 基于堆栈分配器：若游戏是以线性关卡为中心，且内存足够容纳各个完整关卡，则可用堆栈分配器。注意栈顶端先分配驻留资源（LSR，各关卡共享的资源），再分配关卡所需内存。
* 基于池分配器：因为每个内存组块大小相同，要注意设计资源数据时，必须避免大型连续数据结构，容许资源能被切割成同等大小的块。这种分配方式天生的问题就是文件内**最后的组块**空间被浪费。选择组块大小时，可以考虑设为操作系统I/O缓冲区大小的倍数，如512KB。
* 资源组块分配器：专为解决上述组块浪费内存而设的分配模式。只需管理一个链表，内含所有未用满内存的组块以及自由内存块的位置及大小。这种方案有一个问题是卸下资源内存时，其“边角”的组块也会同时消失。解决方案是只利用该种分配器分配**和对应关卡生命期相同的内存**，这需要独立地管理每个关卡的组块，且用户请求分配时指明从哪个关卡分配内存。
* 分段资源文件：将资源文件分为若干段，每段分为若干个组块（与池分配器配合）。各段的作用不同，有的是为主内存而设的数据，有的是仅在载入过程中使用、载入后被弃置的临时数据，有的是发行版本不会载入的调试信息

### 2.2.6 资源的交叉引用

资源的交叉引用意味着资源间的依赖性，所以资源数据库可以表达为依赖对象所组成的有向图。交叉引用可以分为内部（单个文件里对象间的引用）和外部（引用另一个文件的对象）。下图给出了资源数据库的交叉引用例子。

![资源数据库的交叉引用例子](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image%2Fengine%2F%E8%B5%84%E6%BA%90%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9A%84%E4%BA%A4%E5%8F%89%E5%BC%95%E7%94%A8%E4%BE%8B%E5%AD%90.png)

#### 2.2.6.1 处理资源内部引用

在C++中， 由于指针的内存地址总会变，而且离开运行中的程序就失去意义，所以不能用指针来表示对象间的依赖。可以将资源引用存为GUID（全局唯一的字符串或散列码），资源管理器要维护一个全局资源查找表，其中键为GUID，值为资源在内存中的地址。

储存对象到二进制文件的另一常用方法是，把指针转换为为文件偏移值，并建立指针修正表。下图给出了储存二进制文件以及将文件载入内存的指针修正示意，具体过程为：①把每个对象的内存影响遍历一次，顺序写至文件成为连续映像；②写进文件的代码，清楚知道对象的数据类型和类，也就知道每个对象的指针在哪里，把这些指针位置储存到指针修正表并一同写进文件；③载入文件至内存时，映像内对象仍保持连续，并凭借修正表修正所有指针。

![储存二进制文件以及将文件载入内存的指针修正示意](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image%2Fengine%2F%E5%82%A8%E5%AD%98%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%96%87%E4%BB%B6%E4%BB%A5%E5%8F%8A%E5%B0%86%E6%96%87%E4%BB%B6%E8%BD%BD%E5%85%A5%E5%86%85%E5%AD%98%E7%9A%84%E6%8C%87%E9%92%88%E4%BF%AE%E6%AD%A3%E7%A4%BA%E6%84%8F.jpg)

从文件载入C++对象，必须调用对象的构造函数。这个问题有两个常见解决方案：使用纯C结构体来储存数据或使用无虚函数、只含不做事情的平凡构造函数的C++ struct/class；表里记录对象属于哪个类，并使用[placement new](https://en.wikipedia.org/wiki/Placement_syntax)语法调用构造函数，像下面的代码所示。

    void* pObject = ConvertOffsetToPointer(objectOffset);
    ::new(pObject) ClassName;  // placement new语法，ClassName为对象所属的类名

#### 2.2.6.2 处理资源外部引用

以上提及的两个方案，仅对资源内部引用有效。要正确表示外部引用，除了指明偏移值或GUID，还要加上资源对象所属文件的路径。一般做法是：载入每个资源文件时，扫描文件中的交叉引用表，并载入所有被外部引用但未载入的资源文件，当载入所有互相依赖的资源时，就用主查找表把所有指针转换成真实的内存地址。

### 2.2.7 资源载入后初始化

有一些资源载入内存时需要进行一些无法避免的初始化，例如三维网格的顶点和索引载入主内存后，几乎总是要传送至显存，而且只能在运行时进行。在C++中，可以使用多态为每个类设置如`Init()`和`Destroy()`的虚函数用于独立初始化和销毁工作。载入后初始化和资源内存分配策略息息相关，有时初始化会在文件的数据上新增数据（如额外计算类中的成员数据），有时初始化的数据用来取代己载入的数据（如引擎载入过时格式的网格数据，自动转换为最新格式，以保证向后兼容）。可以采用先载入到临时内存区域，初始化完成后再把相关数据复制到内存最终位置。

参考文献：电子工业出版社《游戏引擎架构》第6章
