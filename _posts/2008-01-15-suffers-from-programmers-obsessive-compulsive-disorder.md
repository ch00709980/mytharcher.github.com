---
date: 2008-01-15 01:39:12
layout: post
title: 患上程序员的强迫症加妄想症
category: thinking
tag:
- 强迫症
- 数学
- 程序员
- 算法
---

下午和[erik](http://hi.baidu.com/erik168)同学讨论拖动组件的性能优化问题时，看到他不同的思路，在对需要拖动并排序的情况下，通过对当前拖动模块和各个已存在的同级模块之间的距离计算来完成排序判定，于是回来一直想每次鼠标移动的事件都要计算((x1-x2)2+(y1-y2)2)^2，是否还有优化的余地，于是很夸张的花了20分钟在草稿纸上进行数学演算，以证明是否能通过消去乘方来得到一个与x1+y1和x2+y2的大小有关的近似比较结果，而最简单的数学完全平方公式至少证明了4个事实：

1. 我的猜想是错的；
2. 我花了半天时间证明了不可行性；
3. 我的数学连高中水平都不如了；
4. 我患上了erik所说的程序员的强迫症和妄想症！

强迫自己的去对js这种本来就低效的浏览器程序进行吹毛求疵优化，妄想能通过很微弱的计算量的优化来带来性能的提高。结果erik发来的几行程序给了我重重的一拳：

	var t1 = new Date();
	for(var i = 0; i < 100000; i++) {
		var test = Math.pow(567,2) + Math.pow(567,2);
	}
	alert(new Date() - t1);//500-600 in IE, 400-500 in FF

20w次的的开方运算也就花半秒不到的时间，这样的量级，就算前面那个例子把开方运算去掉，这种优化简直微不足道，在浏览器环境下完全不值一提。而平常总是为了“优化性能”而连循环都反着写：

	for(var i=array.length-1;i>=0;i--){
		//...
	}

目的是好的，但看来我也的确患上了强迫症和妄想症。不过幸得erik同学的点化，js的东西只要不造成循环引导致的内存泄露，稍微注意一些一般是没有问题的，今后少做这种无聊的事情。当然也不能完全不考虑效率，推荐一篇文章“[如何优化JavaScript脚本的性能](http://shiningray.cn/improve-javascript-performance.html)”（终于找到出处），我觉得看后一定会逐渐变得像我现在这样强迫自己做BT的事情。但到了最后写程序还是应该把更多时间投入到架构上，这才是真正能提高境界的地方。

换个思路，克服程序员强迫症。

-EOF-
