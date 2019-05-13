title: Canvas快速入门
date: 2013-01-05 20:13:24
categories:
- 技术
- Web前端
- HTML5
tags:
- HTML5
- Canvas
---
本文是一个Canvas快速入门，亦是[Rob Hawkes《HTML5 Canvas基础教程》人民邮电出版社](http://book.douban.com/subject/7162014/)一书的读书笔记。

<!-- more -->

# 1. Canvas元素

以下html代码定义了一个canvas元素。

    <!DOCTYPE html>
    <html>
    <head>
        <title>Canvas快速入门</title>
        <meta charset="utf-8"/>
    </head>
    <body>
    <div>
        <canvas id="mainCanvas" width="640" height="480"></canvas>
    </div>
    </body>
    </html>

通过以下Javascript语句访问canvas元素：

    //DOM写法
    window.onload = function () {
        var canvas = document.getElementById("mainCanvas");
        var context = canvas.getContext("2d");
    };
    //jQuery写法
    $(document).ready(function () {
        var canvas = $("#mainCanvas");
        var context = canvas.get(0).getContext("2d");
    });
    //接下来就可以调用context的方法来调用绘图API

# 2. 基础API
## 2.1 坐标系统

Canvas 2D渲染上下文采用平面笛卡尔坐标系统，左上角为原点(0,0)，坐标系统的1个单位相当于屏幕的1个像素。具体如下图：  
![HTML5 Canvas坐标系统](http://raytaylorlin-blog.qiniudn.com/image/HTML5/HTML5%20Canvas%E5%9D%90%E6%A0%87%E7%B3%BB%E7%BB%9F.jpg)

## 2.2 绘制基本图形
### 2.2.1 矩形

    //绘制一个填充矩形
    context.fillRect(x, y, width, height)
    //绘制一个边框矩形
    context.strokeRect(x, y, width, height)
    //清除一个矩形区域
    context.clearRect(x, y, width, height)

### 2.2.2 线条
绘制线条与绘制图形有一些区别，线条实际上称为路径。要绘制一条简单的路径，首先必须调用beginPath方法，接着调用moveTo设置路径的起点坐标，然后调用lineTo设置线段终点坐标（可多次设置），再调用closePath完成路径绘制。最后调用stroke绘制轮廓（或调用fill填充路径）。以下为例子：

    //示例
    context.beginPath();    //开始路径
    context.moveTo(40, 40);    //移动到点(40,40)
    context.lineTo(300, 40);    //画线到点(300,30)
    context.lineTo(40, 300);    //画线到点(40,300)
    context.closePath();    //结束路径
    context.stroke();    //绘制轮廓
    //或者填充用context.fill();

### 2.2.3 圆形
Canvas实际上并没有专门绘制圆形的方法，可以通过画圆弧来模拟圆形。由于圆弧是一种路径，所以画圆弧的API应该包含在beginPath和closePath之间。

    /* 绘制一条圆弧
     * @param x,y 圆心的横纵坐标
     * @param radius 圆的半径
     * @param startAngle 起始角度，以弧度为单位，x轴方向为0
     * @param endAngle 结束角度，以弧度为单位
     * @param anticlockwise 布尔值，表示是否按逆时针方向绘制
     */
    context.arc(x, y, radius, startAngle, endAngle, anticlockwise)

更多用法参见[Drawing shapes(MDN)](https://developer.mozilla.org/en-US/docs/HTML/Canvas/Tutorial/Drawing_shapes?redirectlocale=en-US&amp;redirectslug=Canvas_tutorial%2FDrawing_shapes)。

## 2.3 样式
### 2.3.1 修改线条颜色

    var color;
    //指定RGB值
    color = "rgb(255, 0, 0)";
    //指定RGBA值（最后一个参数为alpha值，取值0.0~1.0）
    color = "rgba(255, 0, 0, 1)";
    //指定16进制码
    color = "#FF0000";
    //用单词指定
    color = "red";
    //设置填充颜色
    context.fillStyle = color;
    //设置边框颜色
    context.strokeStyle = color;

### 2.3.2 修改线宽

    //指定线宽值
    var value= 3;
    //设置边框颜色
    context.linewidth = value;

更多样式介绍参见[Applying styles and colors(MDN)](https://developer.mozilla.org/en-US/docs/HTML/Canvas/Tutorial/Applying_styles_and_colors)。

## 2.4 绘制文本

    //指定字体样式
    context.font = "italic 30px 黑体";
    //在点(40,40)处画文字
    context.fillText("Hello world!", 40, 40);

## 2.5 绘制图像
在绘制图像之前，需要先定义图像并加载。

    var img = new Image();
    img.src = "myImage.png";
    img.onload = function () {
        //图像加载完毕执行
    };

以下是drawImage API解释：

    //在(x,y)出绘制图像image
    context.drawImage(image, x, y)
    //在(x,y)出绘制width*height的图像image
    context.drawImage(image, x, y, width, height)
    //在image的(sx,sy)处截取sWidth*sHeight的图像，在(dx,dy)处绘制dWidth*dHeight的图像
    context.drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight)

# 3. 高级功能
## 3.1 使Canvas填满浏览器窗口

最简单的方式是将canvas元素的宽度和高度精确设置为浏览器窗口的宽度和高度，用CSS消去白色空隙。

CSS代码：

    * {
        margin: 0;
        padding: 0;
    }

    html, body {
        height: 100%;
        width: 100%;
    }

    canvas {
        display: block;
    }

Javascript代码：

    function resizeCanvas() {
        //canvas由jQuery获取
        canvas.attr("width", $(window).get(0).innerWidth);
        canvas.attr("height", $(window).get(0).innerHeight);
        context.fillRect(0, 0, canvas.width(), canvas.height());
    }
    $(window).resize(resizeCanvas);
    resizeCanvas();

## 3.2 绘图状态
在canvas中，绘图状图指的是描述某一时刻2D渲染上下文外观的整套属性，包括：变换矩阵、裁剪区域、globalAlpha、globalCompositeOperation、strokeStyle、fillStyle、lineWidth、lineCap、lineJoin、miterLimit、shadowOffsetX、shadowOffsetY、shadowBlur、shadowColor、font、textAlign和textBaseline。

当需要改变画布全局状态时，一般先将当前状态保存起来——调用save方法将状态推入绘图状态栈），做完操作之后，再调用restore方法回复绘图状态。

    //示例
    context.save();
    context.globalAlpha = 0.5;
    context.fillRect(0, 0, 200, 100);
    context.restore();

## 3.3 变形
### 3.3.1 平移
将2D渲染上下文的原点从一个位置移动到另一个位置。注意，这里移动的是坐标原点即全局绘图位置，API如下：

    //将坐标原点移动到(x,y)
    context.translate(x, y)

### 3.3.2 缩放

    //将全局横纵尺寸缩放至x,y倍（即在原有数值乘上倍乘因子）
    context.scale(x, y)

### 3.3.3 旋转

    //将画布绕着原点旋转radius弧度
    context.rotate(radius)

更多高级特性参见[Transformations(MDN)](https://developer.mozilla.org/en-US/docs/HTML/Canvas/Tutorial/Transformations?redirectlocale=en-US&amp;redirectslug=Canvas_tutorial%2FTransformations)。

参考文献：[Rob Hawkes《HTML5 Canvas基础教程》人民邮电出版社](http://book.douban.com/subject/7162014/)