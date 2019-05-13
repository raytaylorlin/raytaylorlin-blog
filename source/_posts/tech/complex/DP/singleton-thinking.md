title: 游戏开发中单例模式的一些思考
date: 2017-01-29 21:27:31
categories:
- 技术
- 综合
- 设计模式
tags:
- 单例模式
- 设计模式
---

往往设计模式的书都是教人如何使用模式，唯独单例模式需要谨慎对待，甚至避免使用。像其他设计决策一样，一旦将一些不必要的单例进行了硬编码，就会带来麻烦。本文描述了单例模式的一些常见问题，以及一些参考解决方案。虽然单例模式存在一些问题，但只要不滥用并仔细思考设计，还是可以享受它带来的好处的。

<!-- more -->

# 1. 单例模式的基础

## 1.1 使用方法

> 单例模式确保一个类只有一个实例，并为其提供一个全局访问入口。

该模式用于一个类如果有多个实例就不能正常运作的情形。最常见的是，这个类与一个**维持着自身全局状态的**外部系统进行交互的情况。像封装底层文件操作API的类，必须知道之前的每一步操作，才能对创建文件和删除文件这些互斥的操作做出正确的反应。

具体实现上，一个单例类一般会遵循这些规则：private构造方法使得对象只能在类的内部实例化，通过public静态属性来访问这个对象，如以下C++代码所示（以封装文件操作API类为例）。

    class FileSystem
    {
    public:
        static FileSystem& Instance()
        {
            // C++11保证一个局部静态变量初始化只进行一次，因此是线程安全的
            static FileSystem* instance = new FileSytem();
            return *instance;
        }
    private:
        FileSystem() {}
    };

## 1.2 模式的特性

* 如果不使用这个类，就不会创建实例
* 在运行时初始化：若是单纯使用静态类而不是单例，则编译器在main函数调用前就自动初始化静态数据了。这意味着**不能使用运行时才知道的信息**（如从文件中加载的配置），也**不能在单例间互相依赖**
* 继承单例是一个强大但经常被忽视的特性：参见下面文件封装类跨平台的例子。


    class FileSystem
    {
    public:
        virtual ~FileSystem() {}
        virtual char* Read(char* path) = 0;
    protected:
        FileSystem() {}
    };

    class PS3FileSystem : public FileSystem
    {
    public:
        virtual char* Read(char* path) { // 调用PS3的文件API }
    }

    class WiiFileSystem : public FileSystem
    {
    public:
        virtual char* Read(char* path) { // 调用Wii的文件API }
    }

    // 关键的跨平台实现
    FileSystem& FileSystem::Instance()
    {
    #if PLATFORM == PLAYSTATION3
        static FileSystem *instance = new PS3FileSystem();
    #elif PLATFORM == WII
        static FileSystem *instance = new WiiFileSystem();
    #endif
    }

# 2. 单例的劣势

* 单例本质是封装到类中的全局变量
    * 令代码晦涩难懂
    * 全局变量促进了耦合：例如全局音频单例类AudioPlayer穿插在各种业务代码中用于播放声音
    * 对并发不友好：多个线程都能访问和修改全局变量，很容易导致死锁，条件竞争或其他难以发现的线程同步的bug
* 单例是个画蛇添足的解决方案：“提供一个全局访问入口”，是使用单例模式的主要原因。像日志类Logger，系统各模块的日志都通过这个单例汇集到一个文件。但这样做的限制是不能创建多个日志器，不能将日志分割为不同的文件。如果后期要修改设计支持多个实例，还需要找出像`Logger.Instance.Write`这样的调用并逐个修改
* 延迟初始化剥离了控制：有的系统初始化时需要耗费一定时间，若在第一次使用时才初始化则可能会引起性能问题。因此在一些对性能有要求的单例中，一般不依赖延迟初始化，即直接像`static Singleton m_instance`这样定义静态变量，在编译时初始化；或者在恰当的时机（如加载界面）去“假调用”令其初始化

备注：选择单例还是静态类，取决于后来是否需要将静态类转换为非静态，前者可以传递实例，而后者需要修改每处调用的代码

# 3. 一些参考解决方案

* 首先考虑究竟是否需要类：游戏中有许多类都是“管理器”，像SoundManager、ParticleManager，包括其他命名如“XXSystem”、“XXEngine”也类似。在设计这样的功能时，要仔细思考一下究竟是否需要类，是否需要单例模式。
* 传递对象到函数中，而不是在函数中调用单例函数：像一些渲染物体的函数需要访问代表图形设备的对象并维护渲染状态，则可能会传递一个context对象到渲染函数中。但另一方面，也要考虑对象是否真的属于某个函数的签名的一部分，像处理AI的函数可能需要写日志，但传递一个Logger对象进去就不合适。
* 将单例的获取限制在继承树中：为了进一步减小单例的“全局影响”，可以考虑在基类（如GameObject）中设置单例，并提供一个protected方法供子类获取示例。
* 通过其他全局对象访问：可能存在代表整个游戏状态的全局Game或World对象，可以考虑将各种全局对象类包装到这种全局类里面，像`Game::Instance.GetAudioPlayer().Play`这样调用。这种做法的好处是如果后续要支持多个Game实例（如用于流处理或测试），则几乎不需要修改其他单例类，副作用就是更多的代码耦合到了Game中。

参考文献：《游戏编程模式》第6章
