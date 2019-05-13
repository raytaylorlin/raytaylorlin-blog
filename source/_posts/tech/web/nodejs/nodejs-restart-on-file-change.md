title: 修改文件后Node.js应用自动重新启动
date: 2013-11-20 09:21:33
categories:
- 技术
- Web前端
- NodeJS
tags:
- Node.js
---
在开发或调试Node.js应用程序的时候，当你修改js文件后，总是要按下CTRL+C终止程序，然后再重新启动，即使是修改一点小小的参数，也总是要不断地重复这几个很烦人的操作。有没有办法做到当文件修改之后，Node.js自动重新启动（或重新加载文件）以节省时间呢？一开始我是想到用grunt的watch模块来监控文件变化，但后来在网上一查，原来我们想到的，别人早已想到，并且已经做得很好。[Node Supervisor](https://github.com/isaacs/node-supervisor)正是这样一个可以实现这种需求的Node.js模块。

<!-- more -->
根据Github上的说明，Node Supervisor原本是用于服务器上Node.js应用崩溃的时候，自己重新启动。当然它也可以监控你的项目的js（或CoffeeScript）文件变化，进而重启来方便我们调试应用程序。

安装方法（以全局模块安装）：
    
    npm install supervisor -g

假设你的Node.js程序主入口是app.js，那么只需要执行以下命令，即可开始监控文件变化。

    supervisor app.js

Supervisor还支持多种参数，列举如下：

    //要监控的文件夹或js文件，默认为'.'
    -w|--watch <watchItems>

    //要忽略监控的文件夹或js文件  
    -i|--ignore <ignoreItems>

    //监控文件变化的时间间隔（周期），默认为Node.js内置的时间
    -p|--poll-interval <milliseconds>

    //要监控的文件扩展名，默认为'node|js'
    -e|--extensions <extensions>

    //要执行的主应用程序，默认为'node'
    -x|--exec <executable>

    //开启debug模式（用--debug flag来启动node）
    --debug

    //安静模式，不显示DEBUG信息
    -q|--quiet

例子：

    supervisor myapp.js
    supervisor -w py_scripts -e 'py' -x python myapp.py
    supervisor -w lib, server.js, config.js, server.js

实现同样功能的类似产品还有[Run.js](https://github.com/DTrejo/run.js)和[Nodeman](https://github.com/remy/nodemon)，这两个我都没用过。但是从文档上来看，前者和Supervisor一样都是极简的5分钟就可以上手的那种，功能比Supervisor稍弱；后者的feature比较多，对应的文档就特别长，估计要研究透也得至少半个小时。选择哪一个，全看项目需求和个人喜好。

参考文献：[Node.js Restart on File Change](http://www.hacksparrow.com/node-js-restart-on-file-change.html)