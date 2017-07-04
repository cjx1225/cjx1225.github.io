---
title: CSS绘制小三角形/圆形等小图标的方法
date: 2017-01-09 21:42:07
categories:
- css
tags: 
- 小三角
- 圆形
- SVG
---
切图时常常需要一些小图标辅助信息模块的表达，比如带有小三角选中状态的导航栏，气泡框等等。如果以图片的方式加载这些小图标会增加http的请求量，不利于性能优化，因此我们可以用CSS，HTML5 Canvas,SVG,基于base64的图片编码等方式实现图标的绘制。
下面主要介绍几种常用的CSS绘制小三角以及其他图形的方法：

## 小三角的绘制

### 利用border属性绘制小三角
**原理**：将原来div的宽和高的值逐渐减为0的过程，每个方向边框的形状会逐渐由梯形变为三角形，设置每个方向边框的颜色和宽度，若只想显示某一块三角形，可以把其他的边框颜色设置为透明，即transparent。
- **边框变换为三角形的几种情况如下：**

```html
	<div class="triangle4"></div> 
	<!--正方形边长为2*border-width 4个三角-->
	<div class="triangle2"></div>
	<!--正方形边长为border-width 2个三角-->
	<div class="triangle3"></div>
	<!--变为矩形长80，宽40 3个三角-->
```

```css
.triangle4
{ width: 0;height: 0;border: 40px;border-style: solid;border-color: #f00 #ff0 #00f #008000;}
.triangle2
{ width: 0;height: 0;border-width: 0 0 40px 40px;border-style: solid;border-color: #f00 #ff0 #00f #008000;}
.triangle3
{ width: 0;height: 0;border-width: 0 40px 40px 40px;border-style: solid;border-color: #f00 #ff0 #00f #008000;}
```
如上所示，分别指定4个，3个，2个方向的border-width宽度，其余方向宽度设为0，将分别得到包含4个三角形的正方形(边长为2*border-width）；包含3个三角形的矩形（一边为border-width，另一边为2*border-width），宽度被设为0的边框对应方向的边框会形成较大的三角形，且长度加倍；包含两个2个三角形的正方形（边长为border-width）。

得到的图形如下所示：
![几种border边框图形](http://img.blog.csdn.net/20161024140654642)
- **得到的几个不同方向的小三角如下：**

```html
	<div class="triangle_up"></div> <!--上三角--> 
	<div class="triangle_down"></div> <!--下三角-->
	<div class="triangle_left"></div>  <!--左三角-->
	<div class="triangle_right"></div> <!--右三角-->
```

```css
.triangle_up{border-color: transparent transparent #00f transparent;}
.triangle_down{border-color: #f00 transparent transparent transparent;}
.triangle_left{border-color: transparent #ff0 transparent transparent;}
.triangle_right{border-color: transparent transparent transparent #008000;}

```
上面的css未列出的其他属性与上述div属性相同，只需更改border-color属性。想得到哪个方向的小三角，就保留哪个方向的border颜色，其他方向的border-color设置为transparent即可。

### css旋转正方形
把一个小的正方形在大的正方形里面旋转45度，背景颜色一样，沿对角线突出一半出去就是三角形。IE9以前不支持旋转。
此种方式不是很简洁，不做过多介绍。

### base64编码
将已有的图片经过base64编码处理，可以有效减少http请求，本文对此不做过多介绍，待后续深入研究。

### HTML5 Canvas
在canvas里面，使用画布标记三个点的坐标绘制小三角形。

```
<canvas id="canTriangle" height="100" width="100">Triangle</canvas>
```
绘制的js代码如下：

```
var canvas = document.getElementById('canTriangle');
var context = canvas.getContext('2d');

context.beginPath();
context.moveTo(0, 0);
context.lineTo(100, 0);
context.lineTo(50, 100);
context.closePath();
context.fillStyle = "rgb(78, 193, 243)";
context.fill();
```
### SVG法
使用内联SVG绘制填充三角形，实现方式如下：

```
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" class="svg-triangle">
 <polygon points="0,0 50,0 25,50"/>
</svg>
```

```css
.svg-triangle
{
    width: 50px;
    height: 50px;
    margin: 20px auto;
}

.svg-triangle polygon
{
    fill: #98d02e;
    stroke: #65b81d;
    stroke-width: 2;
}
```
综上，使用第一种border边框的方式绘制较简单常用，后面两种方式较新，但是功能很强大，有待继续深入研究。

## 其他形状图形的绘制

### 圆形/半圆/弧形绘制

```
	<div class="circle">圆形</div> 
	<div class="semi-circle">半圆</div>  
	<div class="arc">弧形</div>
```

```css
.circle
{
    line-height: 50px;
    width: 50px;
    height: 50px;
    text-align: center;
    border-radius: 50%;
    background-color: red;
}
.semi-circle
{
    width: 50px;
    height: 25px;
    border-radius: 25px 25px 0 0;
    background-color: red;
}
.arc
{
    width: 50px;
    height: 50px;
    -webkit-transform: rotate(45deg);
        -ms-transform: rotate(45deg);
         -o-transform: rotate(45deg);
            transform: rotate(45deg);
    border-radius: 50px 0;
    background-color: red;
}

```
得到的几种图形如图所示：
![这里写图片描述](http://img.blog.csdn.net/20161024144026125)
曲线绘制主要在正方形（矩形）边框的基础上，通过设置border-radius属性设置矩形框4个方向的圆角值：
-圆形是将4个圆角值设为宽度的一半，即圆的半径；
-半圆形将高度减半，上面两个圆角值为宽度的一半，下面两个圆角值设为0；
-弧形保留斜对角方向的两个圆角值为宽度值，另一对角的圆角值设为0，并在此基础上旋转45度可得到图中的效果。旋转功能不兼容IE低版本浏览器。

## 导航栏菜单小三角下标的实现
菜单栏每一项使用相对定位relative，下方对应的小三角使用伪类：after+绝对定位absolute实现，具体实现方式如下：

```
<div class="blog-masthead">
      <div class="container">
        <nav class="blog-nav">
          <a class="blog-nav-item active" href="#">Home</a>
          <a class="blog-nav-item" href="#">New features</a>
          <a class="blog-nav-item" href="#">Press</a>
          <a class="blog-nav-item" href="#">New hires</a>
          <a class="blog-nav-item" href="#">About</a>
        </nav>
      </div>
    </div>
```

```
.blog-masthead { background-color: #428bca;}

.blog-nav-item {
  position: relative;
  display: inline-block;
  padding: 20px;
  font-weight: 500;
  color: #cdddeb;
}
/* Active state gets a caret at the bottom */
//使用伪类添加三角形
.blog-nav .active:after {
  position: absolute;
  bottom: 0;
  left: 50%;
  width: 0;
  height: 0;
  margin-left: -5px;
  vertical-align: middle;
  content: " ";
  border-right: 5px solid transparent;
  border-bottom: 5px solid;
  border-left: 5px solid transparent;
}

```

得到的导航图片如同所示：
![这里写图片描述](http://img.blog.csdn.net/20161024151126695)
 如有其他小图形绘制方法，且待后续分享~

---------
参考资料：
https://segmentfault.com/a/1190000003833676
http://m.blog.csdn.net/article/details?id=50901650

