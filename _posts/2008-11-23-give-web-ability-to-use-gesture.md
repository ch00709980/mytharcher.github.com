---
date: 2008-11-23
layout: post
title: 让Web也拥有华丽的手势操作
category: thinking
tags:
- 手势
---

### 源起iPhone

第一次看到同事用iPhone的时候，我充满了惊奇，因为我从来没想过手机居然可以这样使用，手指在上面划来划去就完成了一些操作。给人一种使用电子产品就像使用真实的原始物理器具般的感觉，这种用户体验实在太酷了！

回到web上，最先想到的是浏览器的手势。在遨游下，按住右键向不同的方向拖放，就会发出不同的快捷命令。我用的最多的就是“向左”和“向右”，也就是“后退”和“前进”。后来得知Firefox也有一款手势插件，于是这个插件就成了我FF必装的几个插件之一。

鼠标手势，相对于键盘按键，鼠标点击来说，是一种模糊的操作概念。键盘敲错一个字结果就完全不同，鼠标也只能点击固定的区域，滚轮也都是按固定的行数来算。但手势就不同了，操作只有一个大概的方向，你不能保证每次向右拖动轨迹的纵坐标都处在同一水平线上。当然，操作越精准，程序越简单，但是我们要的不是让每个用户都精准的操作，因为这样不会为他带来便利，只会增加他对操作学习的成本。就像在滑冰场，要做出花样滑冰大师erik那样的旋转动作，需要极精准的控制力，但像我这样的普通人只想体会简单滑行的乐趣，而并不想在这上面花这么高的成本。不需要很精准的操作让人更容易掌握，正如每个人玩wii都可以很容易上手，而魔兽则不行。

### 识别手势

如把鼠标的移动识别成一个“手势”呢？现在能得到的最多就是只有鼠标移动的轨迹，如何让他变成现实当中的“上下左右”？在我对这个问题如何转化成数学模型一筹莫展的时候，偶然的搜到了一篇，也是当时唯一的一篇有用的文章，[Mouse Gesture Recognition](http://www.bytearray.org/?p=91)里面的这张图真是一图惊醒梦中人啊！这不就是计算机图形学里描述矢量字体图形的方向法吗，现在只要把把所有轨迹中每相邻两点换算成一个方向矢量，这个矢量就代表了每两个点间轨迹的方向。

这样第一个任务就是把所有轨迹描绘的角度放进几个固定的区间，这些区间就是跟据需求划分的方向判定标准。如果是上下左右四个方向，那划归到平面坐标系，就是轴正负方向±π/4的4个区间，也相当于把坐标系的四个象限旋转了-π/4，如图所示：
 
![手势坐标区间](/assets/img/gesture/gesture1.jpg)
 
用一张表来记录鼠标轨迹中这一连串的矢量，不仅包括了方向编码，还有后面会用到的距离值。

接下来第二步要做的是，把这些矢量，根据一定的规则组织起来，最终输出成为手势命令。首先想到的的就是去重，因为同一个方向的多个连续矢量记录一个就可以了，这不是图形学里的字体描绘，多出来的方向码对最终的手势是没有意义的。比如鼠标向右连续滑动的一个动作，并不能看作向右多次移动动作的组合。

顺利的情况下，用户移动鼠标的速度较快，那么鼠标侦测到两个点间的距离就比较长，整个过程会比较流畅，那么输出的手势编码就会比较简单。比如向右很快的拉一条直线应该就输出一个“0”。但是还有大部分情况下很可能用户移动鼠标比较慢，特别是现在的鼠标设计的侦测频率有很大的提高，那么一个动作的移动过程中就有可能侦测到非常多的点。而这么多点连成的线段很有可能不都是用户所希望的操作方向，比如以下的情况：

![手势扰动](/assets/img/gesture/gesture2.jpg)

一个期望正常的“→”的手势，很可能是由一串不正常的“→↑→↓→”所组成。我把这个运动轨迹中个一个波峰（或者波谷）称为一个“扰动”，因为这对程序最终有效的识别手势造成了干扰。于是需要一个策略来识别并消除输出手势编码中的扰动。

我先尝试了对取样点进行时间的限制，相当于人为的让鼠标侦测变慢，比如每50毫秒才记录一次鼠标轨迹。但从实验结果来看，鼠标轨迹本来就是时序上随机的，增加反应时间并不能解决随机性的本质。分析以后发现，可以通过设定一个最小位移的阈值，运动过程中每当出现方向改变时，就判定同个方向累计的移动距离是否超过了这个阈值，如果是，那么就记录这个方向编码，反之就把这个微小的位移判定为一个扰动，并丢弃这个方向编码。其实与遨游和FF的手势判定方式很相似，我设置的位移阈值是20像素，这样基本上屏蔽了很大一部分扰动。

同时，这个位移阈值其实可以看作是手势识别的一个敏感度参数，通过调节这个值的大小来得到不同敏感度的输出命令。比如把值设的大一些，那么判断的“神经”也会比较“大”，从而很长距离的运动才会判定为一个方向编码；反之，如果要侦测更多的扰动细节，就可以把这个值设的小一些。

最终，我们通过鼠标滑动的一个轨迹，计算出了一个用户的手势命令，比如单独的上、下、左、右，画半圆（→↓←），画圈等。然后再将计算出来的手势命令编码对应上要进行的操作，就完成了手势的解码。

### 具体应用

完成了手势的识别，我最先想到的是做一个iPhone那样看照片的小[demo](/demo/gesture/)。例子中为了更说明图片的移动，用了四格的图片框，通过鼠标左键按下后上下左右的移动就可以看到效果。

在Web上，比较适合使用手势操作的就是网络相册的上一张下一张的翻页操作了。在网速达到比较快的情况下，浏览一个相册中的照片，通过左右滑动的手势，会减少用户对“上一页”、“下一页”按钮的寻找时间，浏览起来更加轻松愉快。如果手势能接收到触摸板或者触摸屏的输入信号，那么一切将更完美！

除了相册，目前我能想到可以应用的地方还有Flash游戏，比如奥运游戏中的投标枪，完全可以不通过按住一个按钮等能量槽满释放这样的操作，可以直接就是一个鼠标“扔”的一个手势，方向和力度都可以直接计算。

这样的例子还有很多，可谓是心有多大，舞台就有多大！尤其在Flash程序中，手势的应用前景将会非常的广阔。

### 后记

没想到我们的技术周报刚想尝试下新的形式，却成了一期绝版。虽然我曾预料到这个结果，但没想到来的这么快。昔日的xdjm们正象歌词中唱的“We are a family！”，但马上我们将不再属于同一个部门。不过在往后，我们亲密无间的合作是不会变的，同样不会变的还有：我们的前端组仍会为用户体验的发展提供持续不断的技术原动力！

-EOF-
