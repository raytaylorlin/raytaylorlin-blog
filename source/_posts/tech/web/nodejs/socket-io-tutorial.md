title: Socket.IO入门
date: 2013-11-29 10:50:16
categories:
- 技术
- Web前端
- NodeJS
tags:
- Socket.IO
- Node.js
---
暑假的时候Heatmap项目组需要对在线即时通信和协作进行技术探索，于是我开始研究Web在线聊天的实现方式。在充分对Comet技术进行了研究之后（详见我之前写的一篇[Comet简介的博文](/Tech/other/comet-intro/)），在丁基友的提示之下决定尝试使用Socket.IO。一个是考虑以后HTML5做网络通信需要用到WebSocket现在可以提前接触一下，另外一个是这个东西的服务器端要用到Node.js，之前就对node很有兴趣正好借此提升下功力。

<!-- more -->

# 1. 简介

首先是Socket.IO的官方网站：http://socket.io

官网非常简洁，甚至没有API文档，只有一个简单的“How to use”可以参考。因为Socket.IO就跟官网一样简洁好用易上手。

那么Socket.IO到底是什么呢？Socket.IO是一个WebSocket库，包括了客户端的js和服务器端的nodejs，它的目标是构建可以在不同浏览器和移动设备上使用的实时应用。它会自动根据浏览器从WebSocket、AJAX长轮询、Iframe流等等各种方式中选择最佳的方式来实现网络实时应用，非常方便和人性化，而且支持的浏览器最低达IE5.5，应该可以满足绝大部分需求了。

# 2. 安装部署

## 2.1 安装

首先安装非常简单，在node.js环境下只要一句：

    npm install socket.io

## 2.2 结合express来构建服务器

express是一个小巧的Node.js的Web应用框架，在构建HTTP服务器时经常使用到，所以直接以Socket.IO和express为例子来讲解。

    var express = require('express')
        , app = express()
        , server = require('http').createServer(app)
        , io = require('socket.io').listen(server);
    server.listen(3001);

若不使用express，请参考socket.io/#how-to-use

# 3. 基本使用方法

主要分为服务器端和客户端两段代码，都非常简单。

Server（app.js）：

    //接上面的代码
    app.get('/', function (req, res) {
        res.sendfile(__dirname + '/index.html');});

    io.sockets.on('connection', function (socket) {
        socket.emit('news', { hello: 'world' });
        socket.on('other event', function (data) {
            console.log(data);
        });
    });

首先io.sockets.on函数接受字符串"connection"作为客户端发起连接的事件，当连接成功后，调用带有socket参数的回调函数。我们在使用socket.IO的时候，基本上都在这个回调函数里面处理用户的请求。

socket最关键的是emit和on两个函数，前者提交（发出）一个事件（事件名称用字符串表示），事件名称可以自定义，也有一些默认的事件名称，紧接着是一个对象，表示向该socket发送的内容；后者接收一个事件（事件名称用字符串表示），紧接着是收到事件调用的回调函数，其中data是收到的数据。

在上面的例子中，我们发送了news事件和收到了other event事件，那么客户端应该会有对应的接收和发送事件。没错，客户端代码和服务器正好相反，而且非常相似。

Client（client.js）

    <script src="/socket.io/socket.io.js"></script>
    <script>
        var socket = io.connect('http://localhost');
        socket.on('news', function (data) {
            console.log(data);
            socket.emit('other event', { my: 'data' });
        });
    </script>

有两点要注意的：**socket.io.js路径要写对**，这个js文件实际放在了服务器端的node_modules文件夹中，在请求这个文件时会重定向，因此不要诧异服务器端不存在这个文件但为什么还能正常工作。**当然，你可以把服务器端的socket.io.js这个文件拷贝到本地，使它成为客户端的js文件，这样就不用每次都向Node服务器请求这个js文件，增强稳定性。**第二点是要用`var socket = io.connect('网站地址或ip');`来获取socket对象，接着就可以使用socket来收发事件。关于事件处理，上面的代码表示收到“news”事件后，打印收到的数据，并向服务器发送“other event”事件。

注：内置默认的事件名例如“disconnect”表示客户端连接断开，“message”表示收到消息等等。自定义的事件名称，尽量不要跟Socket.IO中内置的默认事件名重名，以免造成不必要的麻烦。

# 4. 其他常用API
* 向所有客户端广播：`socket.broadcast.emit('broadcast message');`

* 进入一个房间（**非常好用！相当于一个命名空间，可以对一个特定的房间广播而不影响在其他房间或不在房间的客户端**）：`socket.join('your room name');`

* 向一个房间广播消息（发送者*收不到*消息）：`socket.broadcast.to('your room name').emit('broadcast room message');`

* 向一个房间广播消息（*包括发送者*都能收到消息）（**这个API属于io.sockets**）：`io.sockets.in('another room name').emit('broadcast room message');`

* 强制使用WebSocket通信：（客户端）`socket.send('hi')`，（服务器）用`socket.on('message', function(data){})`来接收。

# 5. 使用Socket.IO构建一个聊天室

最后，我们通过一个简单的实例来结束本篇。用Socket.IO构建一个聊天室就是50行左右的代码的事情，实时聊天效果也非常好。以下贴出关键代码：

Server（socketChat.js）

    //一个客户端连接的字典，当一个客户端连接到服务器时，
    //会产生一个唯一的socketId，该字典保存socketId到用户信息（昵称等）的映射
    var connectionList = {};

    exports.startChat = function (io) {
        io.sockets.on('connection', function (socket) {
            //客户端连接时，保存socketId和用户名
            var socketId = socket.id;
            connectionList[socketId] = {
                socket: socket
            };

            //用户进入聊天室事件，向其他在线用户广播其用户名
            socket.on('join', function (data) {
                socket.broadcast.emit('broadcast_join', data);
                connectionList[socketId].username = data.username;
            });

            //用户离开聊天室事件，向其他在线用户广播其离开
            socket.on('disconnect', function () {
                if (connectionList[socketId].username) {
                    socket.broadcast.emit('broadcast_quit', {
                        username: connectionList[socketId].username
                    });
                }
                delete connectionList[socketId];
            });

            //用户发言事件，向其他在线用户广播其发言内容
            socket.on('say', function (data) {
                socket.broadcast.emit('broadcast_say',{
                    username: connectionList[socketId].username,
                    text: data.text
                });
            });
        })
    };

Client(socketChatClient.js)
    
    var socket = io.connect('http://localhost');
    //连接服务器完毕后，马上提交一个“加入”事件，把自己的用户名告诉别人
    socket.emit('join', {
        username: 'Username hehe'
    });

    //收到加入聊天室广播后，显示消息
    socket.on('broadcast_join', function (data) {
        console.log(data.username + '加入了聊天室');
    });

    //收到离开聊天室广播后，显示消息
    socket.on('broadcast_quit', function(data) {
        console.log(data.username + '离开了聊天室');
    });

    //收到别人发送的消息后，显示消息
    socket.on('broadcast_say', function(data) {
        console.log(data.username + '说: ' + data.text);
    });

    //这里我们假设有一个文本框textarea和一个发送按钮.btn-send
    //使用jQuery绑定事件
    $('.btn-send').click(function(e) {
        //获取文本框的文本
        var text = $('textarea').val();
        //提交一个say事件，服务器收到就会广播
        socket.emit('say', {
            username: 'Username hehe'
            text: text
        });
    });

这就是一个简单的聊天室DEMO，你可以根据你的需要随意扩展。Socket.IO基本上就是各种事件的提交和接收处理，思想非常简单。

参考文献：[Socket.IO How to use](http://socket.io/#how-to-use)

参考文献：[Socket.IO官方WIKI](https://github.com/learnboost/socket.io/wiki)



