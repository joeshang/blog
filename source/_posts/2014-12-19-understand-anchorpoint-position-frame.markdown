---
layout: post
title: 理解anchorPoint，position，frame的关系
date: 2014-12-19 21:20:06 +0800
comments: true
categories: iOS-Recipe
---

对于UIView的`frame`，`bounds`，`center`都比较熟悉：

- `bounds`指定UIView自身坐标系。
- `center`指定UIView在superview中的位置坐标。
- `frame`由互不影响（正交）的`bounds`跟`center`生成。

但是我一直有这样的疑问：为什么要使用UIView中心位置的`center`来指定坐标，为什么不用其他位置？

后来学习了CALayer，发现UIView的主要任务其实是响应事件（这就是为什么从UIResponer继承的原因），而将显示委托给CALayer处理。一个成功的UIView背后总有一个默默贡献的CALayer，CALayer作为GPU/OpenGL纹理的一种高层封装，处理显示的方方面面，UIView则将CALayer一些功能再次封装，提供简洁好用的接口。像UIView中的`frame`，`bounds`就是直接使用CALayer中的`frame`，`bounds`。

但是对于`center`，CALayer对应项是`position`，为什么不同名了呢？因为`position`更灵活，谁规定“指定UIView在superview中的位置坐标”一定要在UIView的中心位置呢？！而`position`默认在中心位置的目的我觉得是为了方便Rotation，因为一般Rotation都是绕着中心旋转，UIView为了简化使用，满足大部分情况即可，所以就将默认在中心的`position`封装成了`center`。

由于CALayer的`position`并没有限制一定要在`bounds`的中心位置，所以就需要一个属性来描述`position`在`bounds`中的位置，这样才能推算出`frame`的origin点位置。于是`anchorPoint`出现了，**CALayer用`anchorPoint`指定`position`在`bounds`中的位置**，有如下特点：

- 为了可以任意指定位置，因此`anchorPoint`就不使用绝对值，而是比例值。
- 由于Rotation总是围绕`position`旋转，因此指定`position`在Layer中位置的`anchorPoint`就被命名为**锚点**（像船的锚一样固定位置）。

总之，`frame`是由`position`，`bounds`，`anchorPoint`共同生成（其实还有`transform`，这就需要了解Current Transform Matrix的概念，为了减少复杂性，这里就不涉及了），公式如下：

- frame.origin.x = position.x - bounds.size.width * anchorPoint.x
- frame.origin.y = position.y - bounds.size.height * anchorPoint.y

这就解释了：

1. 为什么anchorPoint改变会导致frame改变，而position却没变？
2. 为什么anchorPoint要使用比例而不是具体值？

豁然开朗。

PS. 多思考，保持好奇心，多从设计者角度想想为什么这样设计，即使是简单的概念，也会有收获。Stay Hungry，Stay Foolish。
