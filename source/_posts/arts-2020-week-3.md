---
layout: title
title: ARTS 2020 Week 3：06.08-06.14
date: 2020-06-14 22:15:04
categories: ARTS
tags:
---
## Algorithm
[Maximum Width of Binary Tree - LeetCode](https://leetcode.com/problems/maximum-width-of-binary-tree/)

还是二叉树相关的题目，不管是否简单与否，按照模块进行训练比较成体系一些。（其实是周末带娃太累，刷不了复杂的题。。。）
计算二叉树的最大宽度这道题本身比较简单，主要有一个思维转换，所谓二叉树的宽度，就是每一层的节点个数，看到层，就转换为二叉树的层序遍历，使用队列，计算每一层节点个数，最后算出最大值即为二叉树的宽度

## Review
[iOS Memory Deep Dive - WWDC 2018 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2018/416/)

WWDC 2020 要来了，看了下 2018 年关于 iOS 内存的一个 Session：
### 内存占用
Pages Memory 和 Page Fault 没什么好说的，OS 基础知识。
iOS 上内存可以分成三类：
1. Clean Memory：可以 Page Out 的内存，例如代码段
2. Dirty Memory：被 App 写入过数据的内存，例如堆、图片解码区
3. Compressed Memory：iOS 设备由于存储硬件的特性，并不会像桌面端一样进行 Swap，而是直接 Page Out。但从 iOS 7 开始，统开始采用压缩内存的办法来释放内存空间，被压缩的内存称为 Compressed Memory，再次访问时会先解压。因此，如果在收到 Memory Warning 时去释放被压缩内存，由于被解压，导致内存用的更多。。。

在一些缓存数据场景，建议用 NSCache 替换 NSDictionary，因为 NSCache 会根据系统情况自动清理内存。

### 内存占用分析工具
![](/images/blog/630C9DF7-3048-40A4-8ECE-41E3AF3D09DB.png)
* malloc_history：查看内存分配历史
* leaks：查看泄漏内存
* vmmap：查看虚拟捏成
* heap：查看堆内存

一些调试技巧：
* Xcode Memory Debugger 可以看内存中所有对象的内存使用情况和依赖关系
* 在 Product -> Scheme -> Edit Scheme -> Diagnostics 打开 Malloc Stack（Live Allocations Only），可以定位占用过大内存

### 图片
图片在使用时，会将 jpg/png/webp 解码成 Bitmap，对于 RGBA，一个像素就是 4 字节，使用建议：
* 使用 UIGraphicsImageRenderer 替代 UIGraphicsBeginImageContextWithOptions，iOS 12 上会自动选择格式，例如黑白图或单色，会讲 RGBA 降为 1 字节。
* 修改颜色，建议用 tintColor，不会有额外的内存开销。
* Downsampling 图片时，一般会先解码，然后搞一个小的画布进行渲染，解码还是造成内存峰尖。因此建议使用 ImageIO 框架，CGImageSourceCreateThumbnailAtIndex 不会造成图片解码。

## Tips
* 如何从 UIBezierPath 中提取构造过程：
[iOS - How to get a list of points from a UIBezierPath? - Stack Overflow](https://stackoverflow.com/questions/3051760/how-to-get-a-list-of-points-from-a-uibezierpath)

* iOS 后台杀 App 以前一直以为只有两种情况：
  * iOS 系统 Bug（13.2 有一波）
  * App 占用内存过多，进入后台后被另一个使用内存大户把系统内存吃光。

最近看同事的分析，发现还会有两点导致系统杀 App：
* 出现内存泄漏：[memory leaks - IOS App getting killed immediately after entering background - Stack Overflow](https://stackoverflow.com/questions/48107801/ios-app-getting-killed-immediately-after-entering-background)
* 进入后台持续访问像 Camera 或者 Shared System Database 等资源，也会导致系统把 App 干掉（[Preparing Your UI to Run in the Background | Apple Developer Documentation](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_background)）

一些第三方 SDK 例如环信会导致这个问题： http://www.easemob.com/question/13822 ，至于环信的原因，有大佬用 Hopper 反解了 EMClient 的 -applicationDidEnterBackground: 方法，如下图。可以看到 isLoggedIn 方法，与登陆了之后才会被 kill 现象完全吻合。

![](/images/blog/4373c66306b22120a4dd0493bcbbb543.png)

至于被 Kill 的原因，是因为其调用了 beginBackgroundTaskWithExpirationHandler，而此方法要求在 expiration 到期前调用 endBackgroundTask，需要成对调用，否则系统会杀掉 App，具体见：
https://developer.apple.com/documentation/uikit/uiapplication/1623031-beginbackgroundtaskwithexpiratio

高级，是时候学一波逆向和 Hopper 了

## Share

[重构技巧：数据选择器与中间层](https://joeshang.github.io/2020/06/14/refactor-multiplexer-and-indirectional-layer/)

