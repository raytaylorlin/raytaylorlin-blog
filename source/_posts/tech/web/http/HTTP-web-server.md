title: HTTP学习笔记：Web服务器
date: 2014-05-04 15:48:05
categories:
- 技术
- Web前端
- HTTP协议
tags:
- HTTP
- Web服务器
---
Web服务器每天会分发出数十亿的Web页面，它是万维网的骨干。Web服务器实现了HTTP和相关的TCP连接处理，管理着Web资源，并负责提供管理功能。本文将一步一步解释Web服务器时如何处理HTTP事务的。

<!-- more -->

Web服务器有各种不同的形式：安装并运行通用的Web服务器软件（如开源的使用最多的Apache），购买预先打包好的软硬件解决方案即Web服务器设备，嵌入式服务器。

通常来说，实际的Web服务器会进行7步（如下图所示）：建立连接，接收请求，处理请求，访问资源，构建响应，发送响应，记录事务处理过程。

![Web服务器请求的基本步骤](http://raytaylorlin-blog.qiniudn.com/image/HTTP/Web%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%AF%B7%E6%B1%82%E7%9A%84%E5%9F%BA%E6%9C%AC%E6%AD%A5%E9%AA%A4.jpg)

# 1. Web服务器基本任务

## 1.1 接受客户端连接

客户端请求一条到Web服务器的TCP连接时，服务器就会建立连接，判断另一端是哪个客户端，将IP地址解析出来。一旦新连接建立起来并被接受，服务器就会将新连接添加到其现存的连接列表中，做好监视连接上数据传输的准备。服务器可以随意拒绝或理解关闭任意一条连接。

可以用“反向DNS”对大部分服务器进行配置，以便将客户端IP地址转换成主机名。Web服务器可以将主机名用于详细的访问控制和日志记录。但要注意的是，书籍名查找可能会花费很长时间，很多大容量的服务器要么会禁止主机名解析，要么只允许对特定内容进行解析。另外，服务器还可以通过*ident协议*找到发起HTTP连接的用户名。

## 1.2 接收请求报文

连接上有数据到达时，服务器会从网络连接中按照以下步骤读取数据（有些Web服务器会用特定的数据结构来存储请求报文）：

* 解析请求行，查找请求方法、URI和版本号，各项由空格分隔，并以CRLF作为行的结束
* 读取以CRLF结尾的报文首部
* 如果有的话，检测以CRLF结尾的标识首部结束的空行
* 如果有的话，读取请求主体（长度由Content-Length首部指定）

Web服务器会不停地观察有无新的Web请求。根据输入/输出结构，Web服务器分为以下几类：

* 单线程：一次只处理一个请求，直到其完成为止。在处理过程中，所有其他连接都会被忽略，这会造成严重的性能问题，只适用于低负荷的服务器。
* 多进程及多线程：可以根据需要为每条连接分配一个线程或进程来处理请求，但当服务器同时要处理成百上千的连接时，可能会消耗太多内存和系统资源，因此应该对线程/进程的最大数量做限制。
* 复用I/O：同时监视所有连接上的活动，当连接状态发生变化时（如有数据可用，或出现错误等等），就对那条连接进行少量的处理，处理接受后将连接返回到开放连接列表中，等待下一次状态变化。
* 复用的多线程Web服务器：总和以上*多线程*和*复用I/O*的优点

## 1.3 处理请求

一旦Web服务器收到了请求，就可以根据方法、资源、首部和可选的主体部分来对请求进行处理。有些方法（如POST）要求请求报文必须带有实体主体部分的数据，有些方法则允许有也允许没有，少数方法（如GET）禁止包含实体的主体数据。

## 1.4 对资源的映射及访问

### 1.4.1 docroot

作为资源服务器，Web服务器负责将请求报文中的URI映射为服务器上适当的内容，然后才发送预先创建好的内容。最简单的资源映射方式是以URI作为名字来访问服务器文件系统中的文件。通常Web服务器有一个特殊的文件夹专门存放Web内容，称为文档的根目录（document root）。**Web服务器从请求报文中获取URI，并将其附加在docroot后面。**服务器要注意不能让相对URL退到docroot之外，将文件系统的其余部分暴露出来。

虚拟托管的Web服务器会在同一台服务器上提供多个站点，每个站点在服务器上都有自己独有的docroot。通过配置Apache的`VirtualHost`块，即使请求的URI完全相同，但docroot不同就可以区分开不同的内容了。

### 1.4.2 目录列表

服务器可以接收对目录URL的请求，并解析成为一个目录而不是文件。可以对服务器进行配置，使其在客户端请求目录URL时采取不同的动作。默认情况下，服务器会去查找目录中名为index.html的文件来代表此目录，也可以通过配置Apache的`DirectoryIndex`来设置默认的要使用的文件名。

如果没有提供默认的index文件，也没有禁止使用目录索引，很多Web服务器都会自动返回一个包含目录列表的HTML文件，可以方便浏览服务器上的文件。可以通过Apache的`Options-Indexes`禁止自动访问目录列表。

### 1.4.3 动态内容资源的映射

Web服务器还可以将URI映射为动态资源，即复杂的后端应用程序（区别于图片、HTML文件等静态资源，动态资源可能是网络摄像机网关、股票交易网关、电子商务网关等等其他应用程序）。

## 1.5 构建响应

一旦Web服务器识别出了资源，就执行请求方法中描述的动作，并返回响应报文。如果事务处理产生了响应主体，服务器要确定主体的[MIME](http://baike.baidu.com/view/160611.htm)类型，将内容放在响应报文中回送过去。可以用文件的扩展名来说明MIME类型。服务器有时也会返回重定向响应而不是成功的报文。有关重定向的说明，可以参照[HTTP状态码维基百科](http://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)中3xx系列状态码说明。

## 1.6 发送响应和记录日志

Web服务器通过连接发送数据时也会面临和接收数据一样的问题，它要记录连接的状态，还要特别注意对持久连接的处理（详情参见[HTTP学习笔记：连接管理 第2节](/Tech/web/HTTP/HTTP-connection-management/)）。最后，当事务结束时，Web服务器会在日志文件中添加一个条目，来描述已执行的事务。更多细节可以参见《HTTP权威指南》第21章。

参考文献：人民邮电出版社《HTTP权威指南》第5章