---
title: CSS三栏自适应布局
date: 2017-01-09 21:42:07
categories:
- css
tags: 三栏布局
---
提到CSS三栏布局，从传统的网页布局角度看，会自然而然的想到盒状模型display + position+ float等属性，本文主要总结几种常见的CSS三栏自适应布局模型，包括基于传统的position和margin属性的布局——绝对定位法，自身浮动法，圣杯布局，双飞翼布局;以及基于css3新特性的弹性盒模型布局:flex布局。
	三栏自适应布局特点：两边栏定宽，中间栏宽度自适应;允许任意列的高度最高

## 绝对定位法

  - 将左右两栏使用absolute定位，因为绝对定位使其脱离文档流，后面的center会自然流动到他们上面，
  - 然后使用margin属性，留出左右元素的宽度，既可以使中间元素自适应屏幕宽度。 
  - 三个div顺序可以任意改变。

不足：如果页面上还有其他内容，top的值需要小心处理，最好能够对css样式进行一个初始化。由于此种方式不常使用，暂不放代码。

## 自身浮动法

 - 对左右两栏分别使用float:left和float:right，float使左右两个元素脱离文档流，中间元素正常在正常文档流中，
 - 使用margin指定左右外边距对其进行一个定位。
 - 中间栏center占据文档流位置，一定要放在最后。

示例代码如下所示：

```html
	<div class="container">
	    <div class="left"></div>
	    <div class="right"></div>
	    <div class="main"></div>
	</div>
	<style>
	.container { 
		background: #eee;
		border: 1px solid #999;
		padding: 20px;
		overflow: auto;
		min-width: 600px;}
	.main { 
		padding: 20px;
		margin-left: 220px;
		margin-right:180px;
		border: 1px solid #999;
		background:white;
		overflow: hidden;}
	.left { 
		width: 160px;
		float: left;
		padding: 20px;
		border: 1px solid #999;
		background:white;}
	.right { 
		width: 120px;
		float: right;
		padding: 20px;
		border: 1px solid #999;
		background:white;}
</style>
```

## 圣杯布局

 圣杯布局是Kevin Cornell在2006年提出的一个布局模型概念。
 - 把主列放在文档流最前面，使主列优先加载；
 - 让三列浮动，左右两栏加上负margin让其跟中间栏div并排;
 - 利用父容器的左、右内边距padding和相对定位realtive实现三栏自适应
 - padding-left值要等于left块的宽度，padding-right值等于right块的宽度
 
```html
//圣杯布局法
<div class="container">
    <div class="main"></div>
    <div class="left"></div>
    <div class="right"></div>
</div>
<style>
.container { 
	padding-left: 210px; 
	padding-right: 190px; 
	min-width: 600px;}
.main { 
	float: left;  
	width: 100%;  
	height: 300px; 
	background-color: rgba(255, 0, 0, .5); }
.left { 
	position: relative; 
	left: -210px; 
	float: left; 
	width: 200px; 
	height: 300px; 
	margin-left: -100%;
	background-color: rgba(0, 255, 0, .5); }
.right {  
	position: relative; 
	right: -190px;  
	width: 180px;  
	height: 300px;  
	margin-left: -180px;
	background-color: rgba(0, 0, 255, .5); }
	</style>
```

## 双飞翼布局
在国内最早由淘宝UED的工程师（玉伯）改进圣杯布局模型并传播开来，命名为双飞翼布局，兼容性好，扩展性强。
 - 把主列放在文档流最前面，使主列优先加载；
 - 让三列浮动，左右两栏加上负margin让其跟中间栏div并排;
 - 利用父容器的左、右内边距padding和相对定位realtive实现三栏自适应
 - 为了main内容不被遮挡，在main里面添加一个子元素center来显示内容，设置content的margin-left和margin-right为左右两栏div留出位置；
 - 主要利用主列的左、右外边距定位。

```html
//双飞翼布局
<h3>使用margin负值法进行布局</h3> 
<div class="container">
        <div id = "main"> 
            <div id = "center"></div> 
        </div> 
        <div id = "left"></div> 
        <div id = "right"></div>
</div>
<style>
.container { 
	border: 2px solid yellow; 
	overflow: hidden; }
#main{ 
	width: 100%;
	height: 100px; 
	background-color: #fff;
	float: left;}
#main #center{ 
	margin:0 210px; 
	height: 100px;
	background-color: #ffe6b8; }
#left,#right{ 
	float: left;
	width: 200px;
	height: 100px;
	background-color: darkorange; }
#left{margin-left: -100%; background-color: lightpink;}
#right{margin-left: -200px;}
</style>
```

圣杯布局和双飞翼布局对比：

 - 双飞翼布局比圣杯布局多使用了1个div，少用大致4个css属性（圣杯布局container的 padding-left和padding-right这2个属性，加上左右两个div用相对布局position: relative及对应的right和left共4个属性；
 - 双飞翼布局子div里用margin-left和margin-right共2个属性，比圣杯布局思路更直接和简洁一点。
 - 在浏览器上的兼容性非常好，IE5.5以上都支持。


----------
## 弹性盒模型布局:flex
弹性布局是指通过调整其内元素的宽高，从而在任何显示设备上实现对可用显示空间最佳填充的能力，布局算法是方向无关的。弹性盒子布局主要适用于应用程序的组件及小规模的布局，而（新兴的）栅格布局则针对大规模的布局。
*兼容性：现代主流浏览器，IE10+*

 - 弹性布局特征：
 - 在外围包裹一层div，设置为display：flex；
 - 中间栏设置flex：1；
 - 盒模型默认紧紧挨着，可以使用margin控制外边距。

```
#box{width:100%;display: flex; height: 100px;margin: 10px;} 
#left_box,#right_box{width: 200px;height: 100px; margin: 10px; background-color: lightpink;} 
#center_box{ flex:1; height: 100px;margin: 10px; background-color: lightgreen}
```

若考虑浏览器兼容问题，可在flex前加上浏览器类型前缀

```
display:-webkit-flex;
display:-moz-flex;
display:flex;
display:-ms-flex;

```

早期版本chrome等浏览器可使用如下box/flexbox属性实现flex布局

```
display:-webkit-box;
display:-moz-flex;
display:flex;
display:-ms-flexbox;

```

以上则是对几种常用三栏式布局模型的总结，对于css弹性盒模型模型的总结略简单，有待后期完善。


参考文章：
> http://www.cnblogs.com/star91/p/5773436.html
