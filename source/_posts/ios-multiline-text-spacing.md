---
title: iOS 如何精确还原 UI 稿多行文字间距
date: 2018-03-29 22:07:15
categories: iOS
tags:
---

![](/images/blog/0BE53420-4644-4FF7-88F3-3A763EF7CFCA.jpg)

关于 iOS 中多行文字行间距这个问题蛋疼了几年了，回忆一下整个历程：

![](/images/blog/A5B7BFA3-CDA4-4217-9258-A0F05781AD02.jpg)

一开始，UI 同学使用 PhotoShop 实现 UI 稿，PhotoShop 的 Label 在相同字体下的高度与 iOS 比就不准，并且使用标注工具进行文字标注时总是紧贴着字形的上下边进行标注，而字体本身有 LineHeight，字形上下是有间距的。为了达到 UI 稿效果，只能用模拟器对着相同尺寸 UI 稿，用标尺工具一点点比较，试出间距值，标注值仅供参考。

![](/images/blog/55A4E1BA-674A-4366-88CD-20A10173C8D6.jpg)

后来 UI 同学换成了用 Sketch 实现 UI 稿，由于 Sketch 使用和 iOS 相同的文本渲染技术，在 Sketch 上新建一个 Label，文本带 LineHeight，有间距，单行文字或文字与其他元素之间的间距终于准确了。

![](/images/blog/4865BF46-43BF-4B9F-BABB-5CBA15B48013.jpg)

但是 Sketch 中处理多行文本时只有 LineHeight 的概念，没有 UILabel 中 LineSpacing 的概念，LineSpacing 只会在行与行中间添加间距，每一行的 LineHeight 保持不变，导致 UI 稿中多行文字修改 LineHeight 之后，用 LineSpacing 并不能完美匹配 UI 稿效果，而且 LineHeight 的变化也会导致文本在和其他控件对齐时与标注对不上。NSParagraphyStyle 虽然有 maximumLineHeight 和 minimumLineHeight 属性，但设置以后是在文本顶部多出间距，而不是上下均匀间距。为了解决这个问题，参考过 [iOS 文本对齐，如何像素般精确还原设计稿](https://zhuanlan.zhihu.com/p/27572662)，使用 Sketch 插件将 LineHeight 修正成 LineSpacing 的效果，但 UI 同学反馈插件不能用，我也没仔细研究如何定制 Sketch 插件，另外，每次用插件修正也比较麻烦，UI 同学存在遗漏的可能性。

![](/images/blog/09A41106-CD15-4581-ADF1-7B4F3A3D9505.jpg)

另外，iOS 的 LineSpacing 一直有个 Bug，一旦中文设置了 LineSpacing，在单行情况下底部会多出 LineSpacing 的间距，多行时就没有这个问题，英文单行也没有这个问题。为了解决这个问题，会判断文字是否超过了一行，如果不超过一行就不设置 LineSpacing。后来嫌麻烦，直接用 baseline 对齐而不是 bottom 对齐，offset 需要加上字体 descent 的大小。

今天偶然看到了 [在iOS中如何正确的实现行间距与行高 - 掘金](https://juejin.im/post/5abc54edf265da23826e0dc9) 这篇文章，豁然开朗。虽然设置 maximumLineHeight 和 minimumLineHeight 会导致显示有偏移，但整体高度是对的，利用 baselineOffset 将偏移修复即可，修复公式为 `(lineHeight - label.font.lineHeight) / 4`。

![](/images/blog/67C28884-0A2E-4E83-9562-D577287C181F.jpg)

经过同 Sketch 对比，与 UI 效果一致。由于设置的是 LineHeight，中文单行文字也没有了底部多出间隔的问题了。最后将相关代码抽成一个 Utils，以后如果 UI 修改了文字的 LineHeight，直接使用这个 Utils 配置 NSAttributedString，就能完美适配 UI 的效果和标注，神清气爽！

```objc
NSMutableParagraphStyle *paragraphStyle = [NSMutableParagraphStyle new];
paragraphStyle.maximumLineHeight = lineHeight;
paragraphStyle.minimumLineHeight = lineHeight;
NSMutableDictionary *attributes = [NSMutableDictionary dictionary];
attributes[NSParagraphStyleAttributeName] = paragraphStyle;
CGFloat baselineOffset = (lineHeight - font.lineHeight) / 4;
attributes[NSBaselineOffsetAttributeName] = @(baselineOffset);
```

一些注意事项：
1. 每种字体的 LineHeight 是不同的，例如 SFUI 的 LineHeight 是字号的 1.2 倍，PingFangSC 的 LineHeight 是字号的 1.4 倍。
2. SFUI 中没有中文字体，最后系统会 fall back 到 PingFangSC，字形的显示是相同的，但是由于字体不用，导致 LineHeight 不一样。用 `systemFontOfSize:size` 和 `fontWithName:@"PingFangSC-Regular" size:size`  设置 UILabel 的 font，相同中文内容的 UILabel 高度不一样。
3. baselineOffset 很奇怪，移动的效果是设置值的两倍，例如设置 1 pt，向上移动 2 pt，所以修复公式最后是 / 4 而不是 / 2。


