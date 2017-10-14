---
layout:     post
title:      利用JS改变html控件位置
category: blog
description: 在js中写一些代码实现自动改变控件位置的动画效果
---

## 利用JS改变html控件位置

我想写一个贪吃蛇小游戏，所以需要完成蛇的自动移动效果，这就需要改变html控件位置。本来我以为实现起来就是像之前写过的类似项目，对位置加加减减就可以实现了，但是没想到踩了很多坑，调了一下午的bug，这里做下总结。

#### 1. 特殊计量单位px

在html里位置的都是使用“px”来计量的，所以我们在获取某一空间位置时需要用下面的代码先转化为int型数：

```
var x = parseInt(els[i].style.left)
```
然后对x进行修改之后，依然要在赋值时加上“px”，否则，位置无法修改，这一点是很多网上的资料没有注意的地方，也有可能是网上的资料对应的版本较老，总之，在这一错误我很长时间都没有发现。当我突然尝试使用“200px”赋值可以对位置直接修改，而使用200赋值不可以直接修改的时候，我才发现这一错误。

代码如下：

```
els = document.getElementsById("block");
var x=parseInt(els.style.left)+20;      //els为获取的对象，left为该对象位置信息，转化为int后进行修改
els.style.left = x + "px";				//对els的left信息进行修改，但注意添加px
```

#### 2. 通过名字获取对象
因为我最开始对蛇的各个模块设置了相同的name，所以不知道该如何获取。但是通过查找资料之后，发现可以获取到一个列表的对象，然后对这一列表遍历，对列表中的对象逐一修改。

示例代码如下：

```
var els =document.getElementsByName("search");
for (var i = 0, j = els.length; i < j; i++){
	alert(els[i].value);
}
```

#### 3. 时间循环实现动画效果
经过查找资料，发现两个函数setInterval()和setTimeout(),第一个函数为时间循环，第二个函数表示时间延迟。这里使用第一个函数可以完成隔多长时间调用某一个函数的效果。

完成自动移动控件动画效果的完整代码如下:

``` javascript
var int=self.setInterval("move()",500)             //每隔500ms调用一次move()函数
function move(){
	var els =document.getElementsByName("block");  //获取所有name为block的对象
	for (var i = 0, j = els.length; i < j; i++){
		var x=parseInt(els[i].style.left)+20;
		els[i].style.left = x + "px";			   //需要加上px
	}
}
```