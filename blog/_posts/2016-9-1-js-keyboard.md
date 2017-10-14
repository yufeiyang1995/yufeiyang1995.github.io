---
layout:     post
title:      关于JavaScript中onkeydown事件
category: blog
description: 初学JavaScript
---

## 关于JavaScript中onkeydown

最近刚刚自学了一下JavaScript，想学完之后尝试做一个贪吃蛇小游戏玩玩，贪吃蛇中我们需要用方向键来进行控制，所以在看到onclick事件的时候就在想键盘的事件响应该怎么做。

一共有三种方法用来处理键盘响应，分别是：onkeydown	某个键盘按键被按下；onkeypress	某个键盘按键被按下并松开；onkeyup	某个键盘按键被松开。

但是我发现教程中给的例子大多数都是对input的输入内容进行响应的，与我想要的东西不太一样。

经过查找资料发现，我的这种想法可以用事件监听来进行实现，代码如下：

``` 
<script>
<!--
	function showkey(){
		key = event.keyCode;
		if (key == 37) alert("按了←键！");
		if (key == 38) alert("按了↑键！");
		if (key == 39) alert("按了→键！");
		if (key == 40) alert("按了↓键！");
	}
	document.onkeydown=showkey;
-->
</script>

```

经过这样处理之后，即可直接对键盘按键进行响应。
