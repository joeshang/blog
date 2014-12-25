---
layout: post
title: UIView中hidden、alpha、clear color与opaque的区别
date: 2014-12-19 21:14:36 +0800
comments: true
categories: iOS-Recipe
---

关于UIView的透视度，有四个属性（下图中红框中的项）都与之相关，作为一个喜欢刨根问底的程（处）序（女）员（座），一定要搞清楚它们各自的用处跟区别是什么呢？

![](../images/blog/2014-12-19-transparency-in-xib.png)

### hidden

此属性为BOOL值，用来表示UIView是否隐藏。关于隐藏大家都知道就是让UIView不显示而已，但是需要注意的是：

- 当前UIView的所有subview也会被隐藏，忽略subview的hidden属性。UIView中的subview就相当于UIView的死忠小弟，老大干什么我们就跟着老大，同进同退，生死与共！
- **当前UIView也会从响应链中移除。**你想你都不显示了，就不用在响应链中接受事件了。

### alpha

此属性为浮点类型的值，取值范围从0到1.0，表示从完全透明到完全不透明，其特性有：

- **当前UIView的alpha值会被其所有subview继承。**因此，alpha值会影响到UIView跟其所有subview。
- **alpha具有动画效果。**当alpha为0时，跟hidden为YES时效果一样，但是alpha主要用于实现隐藏的动画效果，在动画块中将hidden设置为YES没有动画效果。

### backgroundColor的alpha（Clear Color）

此属性为UIColor值，而UIColor可以设置alpha的值，其特性有：

- **设置backgroundColor的alpha值只影响当前UIView的背景，并不会影响其所有subview。**这点是同alpha的区别，Clear Color就是backgroundColor的alpha为1.0。
- **alpha值会影响backgroundColor最终的alpha。**假设UIView的alpha为0.5，backgroundColor的alpha为0.5，那么backgroundColor最终的alpha为0.25(0.5乘以0.5)。

### opaque

此属性为BOOL值。要搞清楚这个属性的作用，就要先了解绘图系统的一些原理：屏幕上的每个像素点都是通过RGBA值（Red、Green、Blue三原色再配上Alpha透明度）表示的，当纹理（UIView在绘图系统中对应的表示项）出现重叠时，GPU会按照下面的公式计算重叠部分的像素（这就是所谓的“合成”）：

> Result = Source + Destination * (1 - SourceAlpha)

Result是结果RGB值，Source为处在重叠顶部纹理的RGB值，Destination为处在重叠底部纹理的RGB值。通过公式发现：当SourceAlpha为1时，绘图系统认为下面的纹理全部被遮盖住了，Result等于Source，直接省去了计算！尤其在重叠的层数比较多的时候，完全不同考虑底下有多少层，直接用当前层的数据显示即可，这样大大节省了GPU的工作量，提高了效率。（多像现在一些“美化墙”，不管后面的环境多破烂，“美化墙”直接遮盖住了，什么都看不到，不用整治改进，省心省力）。更详细的可以读下objc.io中[<绘制像素到屏幕上>](http://objccn.io/issue-3-1/)这篇文章。

那什么时候SourceAlpha为1呢？这时候就是opaque上场的时候啦！当opaque为YES时，SourceAlpha为1。opaque就是绘图系统向UIView开放的一个性能开关，开发者根据当前UIView的情况（这些是绘图系统不知道的，所以绘图系统也无法优化），将opaque设置为YES，绘图系统会根据此值进行优化。所以，如果在开发时某UIView是不透明的，就将opaque设置为YES，能优化显示效率。

需要注意的是：

1. 当UIView的opaque为YES时，其alpha必须为1.0，这样才符合opaque为YES的场景。如果alpha不为1.0，最终的结果将是不可预料的（unpredictable）。
2. opaque只对UIView及其subclass生效，对系统提供的类（像UIButton，UILabel）是没有效果的。

## 参考

- [UIView: opaque vs. alpha vs. opacity](http://stackoverflow.com/questions/8520434/uiview-opaque-vs-alpha-vs-opacity)
- [UIView alpha vs. UIColor alpha](http://stackoverflow.com/questions/20423390/uiview-alpha-vs-uicolor-alpha)
- [UIView的alpha、hidden和opaque属性之间的关系和区别](http://blog.csdn.net/wzzvictory/article/details/10076323)
- [绘制像素到屏幕上](http://objccn.io/issue-3-1/)

