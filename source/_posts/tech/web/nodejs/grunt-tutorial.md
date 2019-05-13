title: 使用Grunt完成前端项目自动化构建工作
date: 2013-11-25 09:21:33
categories:
- 技术
- Web前端
- NodeJS
tags:
- Node.js
- Grunt
- 自动化构建工具
---
Java世界里的Maven提供了强大的包依赖管理和构建生命周期管理。在JavaScript的世界里，随着Node.js的流行，JavaScript原生的构建工具已经成为可能。Grunt的生态系统在迅速的成长，目前已经有上百种插件发布在NPM上可供选择。同时，任何人都可以方便的发布自己的插件到NPM上供其他人使用。通过提供通用的接口以进行代码规范检验（Lint）、合并、压缩、测试及版本控制等任务，Grunt使入门门槛大大降低了。

<!-- more -->

# 1. 关于Grunt

一言以蔽之，Grunt是一个基于Node.js的Javascript任务运行器。当你完成前端项目的开发之后，你需要将代码部署并发布，这时候你可能会重复地执行像压缩，编译，单元测试，代码检查，打包发布的任务。在一个项目过程中，我们不仅要专注于编写代码，还要处理这些重复而枯燥的琐事，实在是不能忍！于是我们需要一个自动化的构建工具，而Grunt就是这样一款可以让我们脱离苦海的自动化构建工具。

你所需要做的只是选择你需要的Grunt插件，配置好Grunt，然后把前端自动化构建的工作交给Grunt来完成即可。

[Grunt官方网站](http://gruntjs.com/)
[Grunt中文网](http://gruntjs.cn/)

# 2. 安装Grunt

1. Grunt和Grunt插件都是通过Node.js包管理器npm来安装和管理的。所以使用Grunt之前，需要先安装[Node.js](http://nodejs.org/)，Grunt 0.4.x要求 0.8.0+的Node.js版本。

2. 接下来是安装grunt-cli。**如果你之前安装过旧版本（0.3.x）的Grunt，需要先`npm uninstall grunt -g`来卸载旧版本的Grunt。** 执行以下语句安装全局的grunt-cli。


    npm install grunt-cli -g

3. 安装grunt-init（可选）。grunt-init是个脚手架工具，它可以帮你完成项目的自动化创建，包括项目的目录结构，每个目录里的文件等。实际项目中往往目录结构已经搭建好，只需要直接配置并使用Grunt，限于篇幅本文就不展开讨论grunt-init的使用方法，有兴趣可以到[这里](http://gruntjs.com/project-scaffolding)了解更多。

# 3. 准备新建Grunt项目

一个标准的配置过程只需要给项目添加两个文件：package.json和Gruntfile.js。

package.json: 该文件用于保存作为npm模块发布的项目元数据。需要在该文件的devDependencies 中列出项目所依赖的Grunt版本和Grunt插件。

Gruntfile: 该文件一般命名为Gruntfile.js或Gruntfile.coffee，用于配置或定义任务和加载Grunt插件。

下面是一个package.json的示例，一般我们就在这上面的基础上修改或添加新的插件依赖即可。

    {
        "name": "my-project-name",
        "version": "0.1.0",
        "devDependencies": {
            "grunt": "~0.4.1",
            "grunt-contrib-concat": "~0.3.0",
            "grunt-contrib-jshint": "~0.6.3",
            "grunt-contrib-uglify": "~0.2.2"
        }
    }

# 4. 选择并安装Grunt插件

Grunt的任务其实就是各种插件的组合使用，你需要的构建功能基本上官网都已经提供了相应的插件，详情可参见[插件列表](http://gruntjs.com/plugins)。下面列举几款常用的插件：

* grunt-contrib-clean: 用于清理指定文件（夹），一般是构建之前或之后进行
* grunt-contrib-concat: 连接源文件，减少HTTP请求
* grunt-contrib-copy: 用于复制文件或目录
* grunt-contrib-jshint: JS代码质量检查工具，类似jsLint
* grunt-contrib-watch: 监视磁盘文件，一旦更改就会重新运行指定的任务，例如使http服务器重新加载源文件
* grunt-contrib-uglify: 压缩JS源文件，提高运行时性能
* grunt-contrib-mincss: 用于CSS压缩
* grunt-contrib-csslint: CSS代码质量检查工具
* grunt-contrib-htmlmin: 用于HTML压缩
* grunt-contrib-imagemin: 用于压缩PNG、JPEG、GIF等图像
......

每个Grunt插件都是一个Node.js模块，都可以用`npm install <module> --save-dev`命令来安装（使用--save-dev参数可以在安装模块的同时将依赖关系添加到package.json中的devDependencies）。

# 5. 配置Gruntfile

安装完插件后，需要配置Gruntfile来添加你所需要的自动化任务。Gruntfile一般由这几部分组成：“wrapper”函数，项目和任务配置，要加载的插件，自定义任务。Gruntfile文件的结构最后应该是类似下面这样四个部分：
    
    // 1. wrapper函数，你的所有Grunt代码都必须写在这个函数里面
    module.exports = function(grunt) {

        // 2. 项目和任务配置
        grunt.initConfig({
            pkg: grunt.file.readJSON('package.json'),
            concat: {
                options: {
                    //定义一个用于插入合并输出文件之间的字符
                    separator: '\n'
                },
                dist: {
                    //用于连接的文件
                    src: ['src/*.js'],
                    //返回的JS文件位置
                    dest: 'release/<%= pkg.name %>.js'
                }
            },
            jshint: {
                //定义用于检测的文件
                files: ['gruntfile.js', 'src/*.js']
            },
            uglify: {
                options: {
                    //生成一个banner注释并插入到输出文件的顶部
                    banner: '/*! <%= pkg.name %> <%= grunt.template.today("dd-mm-yyyy") %> */\n'
                },
                dist: {
                    files: {
                        'release/<%= pkg.name %>.min.js': ['<%= concat.dist.dest %>']
                    }
                }
            }
        });

        // 3. 加载各种任务所需的插件
        grunt.loadNpmTasks('grunt-contrib-concat');
        grunt.loadNpmTasks('grunt-contrib-jshint');
        grunt.loadNpmTasks('grunt-contrib-uglify');

        // 4. 自定义任务
        grunt.registerTask('default', ['jshint', 'concat', 'uglify']);
        grunt.registerTask('concat', [concat', 'uglify']);
    };

上面的代码最后自定义了两个任务，当执行`grunt default`的时候，Grunt就会按照配置依次检查js代码，连接代码，并压缩js，同理当执行`grunt concat`的时候，只会执行最后两个步骤。

最后总结一下，Grunt的使用流程基本上就是：下载需要的插件，配置项目和插件，最后自定义你需要的任务执行各种插件功能。本文只是抛砖引玉，至于更详细的用法，参见比较详细的中文教程[Grunt入门指南](http://gruntjs.cn/getting-started/)。

参考文献：[Grunt打造前端自动化工作流](http://tgideas.qq.com/webplat/info/news_version3/804/808/811/m579/201307/216460.shtml)