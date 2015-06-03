---
layout: post
title: 关于使用CALayer中mask的一些技巧
date: 2014-12-19 21:21:39 +0800
comments: true
categories: iOS-Recipe
---

CALayer拥有`mask`属性，Apple的官方解释如下：

> An optional layer whose alpha channel is used to mask the layer’s content. The layer’s alpha channel determines how much of the layer’s content and background shows through. Fully or partially opaque pixels allow the underlying content to show through but fully transparent pixels block that content.

`mask`同样也是一个CALayer。假设将CALayer本身称为ContentLayer，将`mask`称为MaskLayer，蒙版（Masking）的工作原理是通过MaskLayer的alpha值定义ContentLayer的显示区域：对于ContentLayer上每一个Point，计算公式为`ResultLayer = ContentLayer * MaskLayer_Alpha`。所以当alpha为1时Content显示，alpha为0时Content不显示，其他处于0与1之间的值导致Content半透明。

需要注意的是：

1. MaskLayer的color不重要，主要使用opacity（CALayer中的alpha），但是注意`[UIColor clearColor]`其实就是alpha为0的color。
2. ContentLayer超出MaskLayer以外的部分不会被显示出来。
3. MaskLayer必须是个“单身汉”，不能有sublayers，否则蒙版（Masking）的结果就是未知（Undefined）。

由于`mask`是一个CALayer，可以通过组合产生很多非常棒的效果。例如可以将MaskLayer指定为CAGradientLayer类型实现Gradient效果，可以给MaskLayer添加动画，下面两个例子就是这种用法的经典实例：

- [Facebook Shimmer 实现原理](http://liyong03.github.io/blog/2014/06/01/facebook-shimmer/)
- [基于Core Animation的KTV歌词视图的平滑实现](http://ke.gitcafe.com/2014/10/06/how-to-implement-a-core-animation-based-60-fps-ktv-lyrics-view/)

