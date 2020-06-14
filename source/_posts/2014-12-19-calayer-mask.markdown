---
title: 关于使用 CALayer 中 mask 的一些技巧
date: 2014-12-19 21:21:39 +0800
comments: true
categories: iOS
---

CALayer 拥有 `mask` 属性，Apple 的官方解释如下：

> An optional layer whose alpha channel is used to mask the layer’s content. The layer’s alpha channel determines how much of the layer’s content and background shows through. Fully or partially opaque pixels allow the underlying content to show through but fully transparent pixels block that content.

`mask` 同样也是一个 CALayer。假设将 CALayer 本身称为 ContentLayer，将`mask` 称为 MaskLayer，蒙版（Masking）的工作原理是通过 MaskLayer 的 alpha 值定义 ContentLayer 的显示区域：对于 ContentLayer 上每一个 Point，计算公式为 `ResultLayer = ContentLayer * MaskLayer_Alpha`。所以当 alpha 为 1 时 Content 显示，alpha 为 0 时 Content 不显示，其他处于 0 与 1 之间的值导致 Content 半透明。

需要注意的是：

1. MaskLayer 的 color 不重要，主要使用 opacity（CALayer 中的 alpha），但是注意 `[UIColor clearColor]` 其实就是 alpha 为 0 的 color。
2. ContentLayer 超出 MaskLayer 以外的部分不会被显示出来。
3. MaskLayer 必须是个“单身汉”，不能有 sublayers，否则蒙版（Masking）的结果就是未知（Undefined）。

由于 `mask` 是一个 CALayer，可以通过组合产生很多非常棒的效果。例如可以将 MaskLayer 指定为 CAGradientLayer 类型实现 Gradient 效果，可以给 MaskLayer 添加动画，下面两个例子就是这种用法的经典实例：

- [Facebook Shimmer 实现原理](http://liyong03.github.io/blog/2014/06/01/facebook-shimmer/)
- [基于 Core Animation 的 KTV 歌词视图的平滑实现](http://ke.gitcafe.com/2014/10/06/how-to-implement-a-core-animation-based-60-fps-ktv-lyrics-view/)



