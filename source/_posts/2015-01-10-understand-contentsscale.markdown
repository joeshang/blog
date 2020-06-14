---
title: 理解 contentsScale
date: 2015-01-10 01:33:10
comments: true
categories: iOS
---

最近在看《iOS CoreAnimation: Advanced  Techiques》时，不太理解 CALayer 的 `contentsScale` 属性，在后面的 CATextLayer 中再次遇到，于是花功夫 Google 了一下各类文档，下面说说自己对 `contentsScale` 的理解，可能涉及的方面有些多：

## Point与Pixel

iOS 中的绘图系统使用的尺寸单位为 Point，而屏幕显示的单位为 Pixel，为什么要这样做呢？其实就是为了**隔离变化**：对于绘图而言，并不关心如何在屏幕上显示，这些属于硬件细节，也不应该关心，因此框架使用了万金油方法——抽象，绘图使用与硬件无关的 Point，系统根据当前屏幕的情况自动将 Point 转成 Pixel，所以不论以后硬件屏幕如何变化，使用 Point 的绘图系统以不变应万变，这就是抽象的好处！

对于非 Retina 与 Retina 的屏幕，Point 与 Pixel 的转化关系如下：

- 非 Retina：1 Point = 1 x 1 Pixel
- Retina：1 Point = 2 x 2 Pixel

系统通过一个变量来表示这种映射关系，这就是 UIScreen 中的 `scale` 属性的作用。需要注意的是，Retina 屏幕 1 个 Point 对应 4 个 Pixel，但 `scale` 为 2，为什么不是4呢？因为在屏幕的二维空间中，一切可显示的物体都有 X 轴跟 Y 轴性质，Retina 屏幕的映射是 X 跟 Y 两个方向同时放大两倍实现的，所以 `scale` 为 2（这也是为什么上面写成 2 x 2 而不是直接为 4 的原因）。

## CALayer 与 Render

在 iOS 中，如果想要显示什么，可能第一时间想到的就是 UIView 及其子类，但是 UIView 本身其实并不负责显示，从 UIView 从 UIResponder 继承可以看出，UIView 的主要任务是响应触摸事件（在责任链中），那 UIView 是如何实现显示的呢？通过**组合**，将责任委托给成员变量：每个 UIView 都有一个 Backing Layer，UIView 将显示的任务就交给 CALayer 这个小弟啦，自己作为一个 Wrapper，将 CALayer 一些比较复杂的操作封装成简单已用的接口，供外部使用。下图就是二者的关系图（直接用笔记中的手绘，见谅）：

![](/images/blog/calayer-uiview-structure.jpg)

因此对纹理封装的 CALayer 才是显示的核心，CALayer 的 `contents` 才指定了真正的要显示的内容，理解了这一点，下面就开始介绍正主`contentsScale`（现在才开始真是醉了），按照惯例，先看下官方文档，我截取了一段：

> This value defines the mapping between the logical coordinate space of the layer (measured in points) and the physical coordinate space (measured in pixels). Higher scale factors indicate that each point in the layer is represented by more than one pixel at render time. 
Core Animation uses the value you specify as a cue to determine how to render your content.

官方文档有两个关键点：

1. `contentsScale` 决定了 CALayer 的内容如何从 Point 映射到 Pixel。
2. `contentsScale` 决定了 CALayer 的内容如何被渲染。

为了理解这两个关键点，就要介绍下 CALayer 内容的来源了。CALayer 内容的来源有两种：

### 1. 通过 Core Graphics 自定义绘图到 CALayer 的 Backing Store 中

CALayer 会根据 `contentsScale` 创建 Backing Store，并且根据 `contentScale` 设置 Context 的 CTM（Concurrent Transform Matrix）。例如，Layer 的尺寸为 10 Point x 10 Point，当`contentsScale` 为 2.0 时，最后生成的 Backing Store 大小为 20 Pixel x 20 Pixel，并且在将创建的 Graphics Context 传入 `drawLayer:InContext:` 前，会调用 `CGContextScaleCTM(context, 2, 2)` 进行放大，这样会使生成的内容自动满足屏幕的要求。

### 2. 将CGImage类型的图片设置为CALayer的contents

图片的尺寸是像素相关的，iOS 在加载图片时，通过图片的名称进行处理。例如对于 @2x 的图片，在加载为 UIImage 时，会将 `scale` 设置为 2，`size` 为像素大小除以 `scale`，这样就保证了 UIImage 在 Retina 屏幕上能够正常显示。但需要注意的是：将 UIImage 转换成 CGImage 时会丢失 `scale` 属性，使用 CGImageGetWidth/Height 时得到的是像素尺寸。

与 Core Graphics 生成内容不同的是，图片作为纹理已经被上传至 GPU，CALayer 不需要分配 Backing Store，`contentsScale` 会在渲染时起作用：对于 Retina 的屏幕，如果 `contentsScale` 为 2.0，与屏幕的 `scale` 匹配，则渲染系统不对内容进行处理，如果 `contentsScale` 为 1.0，说明 Layer 的内容并不匹配屏幕 `scale`，渲染系统会对 Layer 的内容进行两倍的 Scale 操作。由于图片本身内容没那么多，于是渲染系统会填充像素，导致模糊。

总之，`contentsScale` 描述了 CALayer 内容的 Scale 特性，CALayer 在生成内容时会根据 `contentsScale` 做处理，并且渲染系统根据 `contentsScale` 进行渲染。另，《编写可读代码的艺术》中写到的“名字才是最好的注释”在这里得到很好的诠释，理解了 `contentsScale` 之后，再回过头看看命名，清晰准确，值得学习。

## 需要注意 scale 的地方

1. CALayer 的 `contentsScale` 默认为 1.0，只有在使用 Core Graphics 在 `drawRect:` 中自定义绘图时系统才会根据当前屏幕的情况设置，因此以下情况需要设置合适的 `contentsScale`：
  - 直接设置 CALayer 的 `contents` 时。
  - 创建新的 CALayer 时。例如，CATextLayer 在 Retina 屏幕时如果不设置 `contentsScale`，所显示的文字就会模糊。
2. 在使用 Image Context 时需要注意：`UIGraphicsBeginImageContext` 以 1.0 的比例系数创建 Bitmap，所以当屏幕为 Retina 时，在渲染时可能会显得模糊。要创建比例系数为其它值的图片，需要使用 `UIGraphicsBeginImageContextWithOptions`。

## 参考

- [When do I need to set the contentsScale property of a CALayer?](http://stackoverflow.com/questions/18459078/when-do-i-need-to-set-the-contentsscale-property-of-a-calayer)
- [何时需要考虑 scale 系数](http://edsioon.me/when-consider-scale/)
- [UIView's contentScaleFactor depends on implementing drawRect:?](http://stackoverflow.com/a/26054122/1971624)

