title: Node.js学习笔记：进程与集群
date: 2014-11-11 10:24:05
categories:
- 技术
- Web前端
- NodeJS
tags:
- Node.js
- 进程
---

Node中的Javascript运行在单进程单线程上带来了很多好处：程序状态单一，没有多线程的锁、线程同步问题，操作系统调度因为较少的上下文切换开销，可以很好地提高CPU的使用率。但是这种模型并非是完美的，尤其是如今CPU基本都是多核的，一个Node进程只能利用一个核。此外，一旦单线程上抛出的异常没有被捕获，将会引起整个进程的崩溃。

本文将叙述Node如何应对“如何充分利用多核CPU服务器”及“如何保证进程的健壮性和稳定性”这两个问题。

<!-- more -->

# 1. 服务器模型的变迁

Web服务器的架构至今已经历了几次变迁：

1. 同步：最早的服务器的执行模型是同步的，其一次只为一个请求服务，其余请求都处于耽误的状态。这类架构如今已基本淘汰，只在一些无并发要求的应用中存在。
2. 复制进程：每有一个连接，就复制一个进程来提供服务。这个模型不具备伸缩性，一旦并发请求过高，内存会随着进程数的增长耗尽。
3. 多线程：类似多进程模式，对每一个连接都创建一个线程去服务。线程相对进程开销要小很多，而且线程间可以共享数据。但是多线程还是会随着并发数的增多而耗尽内存，缺乏强大的伸缩性。
4. 事件驱动：单线程的事件驱动避免了不必要的内存开销和上下文切换开销，不受资源上限的影响，伸缩性远比前两者高。

# 2. 多进程架构

## 2.1 创建子进程

面对单进程单线程对多核利用不足的问题，前人的经验是启动多个进程即可，理想状态下每个进程各自利用一个CPU。Node提供的child_process模块的`fork()`、`spawn()`、`exec()`、`execFile()`函数可以实现子进程的创建。以下代码会根据当前机器上的CPU数复制（fork）出对应的Node进程。

    /* master.js */
    var fork = require('child_process').fork;
    var cpus = require('os').cpus();
    for (var i = 0; i < cpus.length; i++) {
        fork('./worker.js');    // worker.js为启动HTTP服务器的代码
    }

这就是著名的Master-Worker（主从）模式，是典型的分布式架构中用于处理业务的模式，具备较好的可伸缩性和稳定性。主进程只负责调度或管理工作进程，工作进程只负责具体的业务处理。

## 2.2 进程间通信

主从进程通过`send()`和`message`事件实现进程间通信，如下面的代码所示：

    /* master.js */
    var subProc = require('child_process').fork('./worker.js');
    subProc.send({hehe: '123'});
    subProc.on('message', function(msg) {
        console.log('MASTER got message:', msg);
    });

    /* worker.js */
    process.on('message', function(msg) {
        console.log('WORKER got message:', msg);
    });
    process.send({foo: 'bar'});

主从进程之间的通信实际上通过IPC（Inter-Process Communication）通道来传递信息的。Node中实现IPC通道的具体细节由libuv提供，在Windows下由命名管道实现，\*nix系统采用Unix Domain Socket实现。**父进程在创建子进程之前，会创建IPC通道并监听它，然后才真正创建子进程，并通过环境变量NODE_CHANNEL_FD告诉子进程这个IPC通道的文件描述符。子进程在启动的过程中，根据文件描述符去连接这个IPC通道，从而完成父子进程之间的链接。**

## 2.3 句柄传递

通常如果让多个进程监听同一个端口，会抛出EADDRINUE异常。要解决多进程监听同个端口，其中一种做法是主进程监听主端口（如80），对外接收所有网络请求，再分别代理到不同端口的进程上。这样既能监听同个端口，甚至可以在代理进程上做适当的负载均衡，缺点是会浪费掉一倍数量的文件描述符。

为了解决上述问题，Node在版本v0.5.9引入了进程间发送句柄的功能，`send()`方法的第一个参数是要发送的数据，第二个可选参数就是句柄。目前可以发送的句柄包括：net.Socket、net.Server、net.Native、dgram.Socket、dgram.Native。

    /* master.js */
    var cp = require('child_process');
    var child1 = cp.fork('./worker.js');
    var child2 = cp.fork('./worker.js');

    var server = require('net').createServer();
    server.listen(1337, function() {
        child1.send('server', server);
        child2.send('server', server);
        // 关掉服务器是关键
        server.close();
    });

    /* worker.js */
    var server = require('http').createServer(function(req, res) {
        res.writeHead(200, {'Content-Type': 'text/plain'});
        res.end('Handled by child, pid = ' + process.pid + '\n');
    });
    process.on('message', function(msg, tcp) {
        if (msg === 'server') {
            tcp.on('connection', function(socket) {
                server.emit('connection', socket);
            });
        }
    });

启动master.js，每次请求`http://localhost:1337`时，得到的都是可能不一样的pid进程的响应，所有请求都由子进程来处理了。要特别注意的是，上述代码看似把`server`对象发送到了子进程，实际上传递的只是文件描述符和消息，子进程根据message.type创建对应的服务器对象， 然后监听到文件描述符上。

# 3. 构建稳定的集群

前面搭建集群的方法充分利用了多核CPU资源，但每个工作进程依旧是在单线程上执行的，它的稳定性还不能得到完全的保障，需要建立一个健全的机制来保障Node应用的健壮性和稳定性。

## 3.1 进程事件

子进程对象除了message事件外，还有一些表示异常或错误的事件：

* error：子进程无法被复制创建、无法被杀死、无法发送消息时触发
* exit：子进程退出时触发
* close：在子进程的标准输入输出流中止时触发
* disconnect：调用`disconnect()`方法时触发，调用该方法会关闭IPC通道

除了上一章中提到的`send()`方法外，还能通过`kill()`方法给子进程发送消息，该方法并不是真正将子进程杀死，而是给子进程发送一个系统信号SIGTERM，子进程收到后才做出约定的行为，如退出进程。

## 3.2 自动重启

可以通过监听子进程的exit事件来获知其退出的信息。当因为有bug导致工作进程退出，需要仔细处理这种异常，最好是工作进程在得知自己要退出时，向主进程发送一个“自杀信号”，然后才停止接收新的连接，当所有连接断开后才退出。

    process.on('uncaughtException', function(err) {
        logger.error(err);    // 记录日志，因为出现未能捕获的异常是不合格的
        process.send({act: 'suicide'});
        // 停止接收新的连接
        woker.close(function() {
            // 所有已有连接断开后，才退出进程
            process.exit(1);
        });
        // 设置超时时间，专门应对长连接这种情况
        setTimeout(function() {
            process.exit(1);
        }, 5000);
    });

此时主进程要监听message事件，一旦接收到自杀信号，就要重新启动一个新的工作进程来顶替出异常即将退出的进程，这样就能使应用平滑地应对用户的请求。工作进程也不能无限地被重启，因为如果在启动的过程或启动后收到连接就发生了错误，会导致工作进程被频繁重启，所以应该有一种机制来限制单位时间内重启的次数，超过限制就触发giveup事件告知主进程放弃重启。通常来说，giveup是比uncaughtException更严重的异常，必须严格监控并避免。

## 3.3 状态共享

Node进程不宜存放太多数据，因为这会加重垃圾回收的负担，同时Node也不允许多个进程间共享数据。在实际业务中，解决数据共享最直接简单的方案就是使用第三方存储，如数据库、磁盘文件、缓存服务（Redis等）。如果采用这种方式，需要一种机制在数据发生改变时通知到各个子进程。一种方式是子进程去向第三方定时轮询；另一种方式是额外增加一个通知进程来轮询第三方，当有数据变化时主动通知所有子进程。主动通知机制如果按进程间信号传递，在跨多台服务器时会失效，所以可以考虑采用TCP或UDP的方案。

## 3.4 Cluster模块

前文从原理层面介绍了child_process模块的一些细节，事实上Node在v0.8版本新增的cluster模块提供了更简洁强大的API来解决上述问题，详情可以参见[Node cluster API文档](http://nodejs.org/api/cluster.html)，此处不再赘述。

参考资料：《深入浅出NodeJS》第九章