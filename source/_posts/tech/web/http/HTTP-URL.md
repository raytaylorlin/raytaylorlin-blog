title: HTTP学习笔记：URL与资源
date: 2014-03-31 9:15:24
categories:
- 技术
- Web前端
- HTTP协议
tags:
- HTTP
- URL
---
URL（Uniform Resource Locator ，统一资源定位符）是浏览器寻找信息时所需的资源位置。通过URL这种因特网的标准化名称，人类和应用程序才能找到并使用因特网上大量的数据资源。本文将介绍URL的语法，Web客户端支持的URL快捷方式，URL编码和字符规则等等。

<!-- more -->

服务器资源名被称为URI（统一资源标识符），而我们常说的URL，实际上是URI的一个子集，也是URI最常见的形式。除此之外，URI还包括URN（其通过名字来识别资源，与它们当前所处位置无关，现仍处于试用阶段）。实际上HTTP应用程序处理的只是URL，所以下面讲的基本都是URL。

# 1. URL的语法

大多数URL语法都建立在以下9部分构成的通用格式上，其中最重要的方案（scheme）、主机（host）和路径（path）：

    <scheme>://<user>:<password>@<host>:<port>/<path>;<params>?<query>#<frag>

1. 方案（scheme）：它会告诉负责解析URL的应用程序应该使用什么协议，其大小写无关。一般有http、https、ftp、mailto、telnet等等。例子：`http://www.baidu.com`
2. 主机与端口（host、port）：主机标识了因特网上能够访问资源的宿主机器，可用主机名（域名）或IP地址表示；端口标识了服务器正在监听的网络端口，对下层使用TCP协议的HTTP来说，默认端口号为80。例子：`http://115.156.216.106:3000`
3. 用户名与密码（user、password）：有一些服务器需要用户输入用户名和密码才允许访问数据。若URL是FTP协议而没有指定这两者，浏览器会自动插入“anonymous”和一个默认密码。例子：`ftp://anonymous:my_passwd@ftp.prep.ai.mit.edu/pub/gnu`
4. 路径（path）：说明了资源位于服务器的什么地方，通常像一个分级的文件系统路径。例子：`http://localhost:3000/css/common.css`
5. 参数（params）：向应用程序提供它们所需的输入参数，以便正确地与服务器进行交互，形式为key-value对列表，由“;”将其与URL其余部分分隔开来。例子：`ftp://prep.ai.mit.edu/pub/gnu;graphics=true`
6. 查询字符串（query）：可以通过查询字符串来缩小所请求资源的范围，形式同样为key-value对，之间用字符“&”分隔，由“?”将其与URL其余部分分隔开来。例子：`http://localhost/test?id=123&show=false`
7. 片段（frag）：表示一个资源内部的片段，通常用于在页面中设置“书签”并实现页内跳转。片段出现在URL的最右边，前面有一个字符“#”。**注意客户端不会将片段发送到服务器，浏览器从服务器获得整个资源后，会根据片段在页内跳转到指定的位置。**例子：`http://localhost/test#hehe`

# 2. URL快捷方式

URL有两种方式：绝对的和相对的。像上面列举的都是绝对URL，包含了访问资源所需的全部信息。相对URL是一种简写方式，需要相对一个**基础URL**进行解析。

相对URL到绝对URL的转换处理，首先是要找到基础URL，一般可以显示提供（比如HTML文档定义一个<base>标签显式指定基础URL），或者在封装资源中提供（比如HTML文档中的a标签链接，其基础URL就是这个HTML文档本身）。接着就是通过以下算法把相对URL转换成绝对URL。

![相对URL转换成绝对URL算法流程图](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/HTTP/%E7%9B%B8%E5%AF%B9URL%E8%BD%AC%E6%8D%A2%E6%88%90%E7%BB%9D%E5%AF%B9URL%E7%AE%97%E6%B3%95%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg)

# 3. URL字符与编码

通常来说，URL采用的是US-ASCII字符集，但是这有很多局限性，因此用一种“转义”表示法来表示不安全字符，通过这种编码机制来避开各种限制。这种转义表示法包含一个百分号“%”，后面跟着两个表示ASCII字符的十六进制数。例如URL中的“~”编码成“%7E”，空格编码成“%20”，“%”编码成“%25”。此外URL还有一些字符用作保留字符，如`%/.#?;:@&=`等等，此处不再赘述。

URL是一种强有力的工具，可以用来命名所有现存对象，也可以很方便地包含一些新格式。但它并不完美，它们表示的是实际的地址而不是准确的名字，这就意味着如果资源被移走了，URL也就失效了（404 not found）。URN就是为了应对这种情况的，无论对象搬移到什么地方，URN都能为对象提供一个稳定的名称。当然，URN背后的思想已经提出一段时间了，但是从URL转换成URN是一项巨大的工程，标准化工作的进程非常缓慢，所以现在因特网资源仍以URL来命名，而且这种趋势仍会保持相当长一段时间。

参考文献：人民邮电出版社《HTTP权威指南》第2章