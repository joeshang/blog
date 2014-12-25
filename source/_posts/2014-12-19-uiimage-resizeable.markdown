---
layout: post
title: UIImage图片拉伸
date: 2014-12-19 21:09:29 +0800
comments: true
categories: iOS-Recipe
---

## 为什么进行图片拉伸

1. 灵活：对于一些带边框的背景图而言，如果不进行图片拉伸，每当改变frame时就需要一份新size的背景图
2. 节约资源：对于一些比较规则的图片，没必须使用frame大小，而是使用一个满足拉伸条件的较小的图片进行拉伸，这样加载图片变快，而且图片大小变小

## 使用方法

使用`- resizeableImageWithCapInsets:(UIEdgeInsets)capInsets`，iOS官方解释如下：

> You use this method to add cap insets to an image or to change the existing cap insets of an image. In both cases, you get back a new image and the original image remains untouched.
During scaling or resizing of the image, areas covered by a cap are not scaled or resized. Instead, the pixel area not covered by the cap in each direction is tiled, left-to-right and top-to-bottom, to resize the image. This technique is often used to create variable-width buttons, which retain the same rounded corners but whose center region grows or shrinks as needed. For best performance, use a tiled area that is a 1x1 pixel area in size.

原理就是对UIImage指定了Insets后，在Insets中的区域不进行拉伸，只对区域外的部分进行拉伸。需要注意的有两点：

1. `UIEdgeInsetsMake`的四个参数从前到后依次是top、left、bottom、right，不是成对的，别搞错了
2. 拉伸的方式**不是全方向一起拉伸！而是先左右拉伸，再上下拉伸，最后的结果是两次拉伸的叠加！**不然会出现[这样的问题](http://stackoverflow.com/questions/18605514/how-to-use-uiimage-resizableimagewithcapinsets)。

## 参考

- [[UIImage resizableImageWithCapInsets:]使用注意](http://www.cnblogs.com/scorpiozj/p/3302270.html)
- [带边框的UIImage缩放](http://onevcat.com/2011/12/uiimage/)
