---
layout: post
title: SCRAnimationChain的设计思路
date: 2014-12-22 11:45:56 +0800
comments: true
categories: iOS-Architecture
---

## 1. 出现原因

自从块（Block）方式的`animateWithDuration:delay:options:animations:completion:`方法出现之后，大家就使用此方法替换原来笨重且不清晰的旧方法来实现动画。但是在实现**动画序列**的时候，问题来了，看下面的代码：

```objc
[UIView animateWithDuration:0.25 animations:^{
    self.imageView.alpha = 0.0;
} completion:^(BOOL finished) {
    if (finished) { 
        [UIView animateWithDuration:0.25 animations:^{
            self.headline.alpha = 0.0;
        } completion:^(BOOL finished) {
            if (finished) {
                [UIView animateWithDuration:0.25 animations:^{
                    self.content.alpha = 0.0;
                } completion:^(BOOL finished) {
                    if (finished) {
                        [UIView animateWithDuration:0.25 animations:^{
                            self.headline.alpha = 1.0;
                        } completion:^(BOOL finished) {
                            if (finished) {
                                [UIView animateWithDuration:0.25 animations:^{
                                    self.content.alpha = 1.0;
                                }];
                            }
                        }];
                    }
                }];
            }
        }];
    }
}];
```

为了实现动画序列，需要在前一个动画的completion块中新建下一个动画，当动画比较多的时候，嵌套层数能把人看醉了，维护起来各种蛋疼。作为一个有重度代码洁癖的程（处）序（女）员（座），怎么能够忍受这样的代码？所以就想实现一种动画的容器，将嵌套关系转换成并列关系，利用递归来封装嵌套操作，将动画序列用简单清晰的方式管理起来。

## 2. 设计思路

在实现之前，按照经验，先去Github上找一找有没有别人已经做好的实现，没必要重新造轮子，而且不一定造出来的有别人的好（多么痛的领悟~）。既然是开源，把别人的看懂了不就相当于自己的嘛。而且，一般你想到的自己觉得很犀利的点子说不定人家已经做烂了（多么痛的领悟x2~）。果然，找到了下面两个库，于是我读了下他们的源码，分析如下：

- [UIKitAnimationPro](https://github.com/demon1105/UIKitAnimationPro)：将动画抽象成Action，用类似Spirit Kit的样式提供动画序列，思路很棒，但是问题在于：
  * 思路是为某一个UIView指定动画序列，这样不是特别合理，因为动画序列很有可能包含多个UIView的动画。
  * 虽然将rotate，scale等操作封装起来方便使用，但是还是不够灵活，应该提供options的接口。
  * 只能串行，没有并行。
- [CPAnimationSequence](https://github.com/yangmeyer/CPAnimationSequence)：没有上述的缺点，已经是非常全面的实现了。但是在读源码的过程中发现里面使用了组合（Composite）模式，但是并没有用好。基类不是一个接口类（Component），而是一个具体的节点类（Leaf），容器类（Composite）继承节点类。这里不是教条主义，一定要按照设计模式要求的来，而是有以下的问题：
  * 由于Composite继承了Leaf，导致Composite中从Leaf继承下来一些跟Composite无关的数据，只好使用NSAssert来确保外部不会使用，这是不合理的。
  * 并没有将组合模式的优点发挥出来，导致代码不够清晰。

基于以上原因，我决定实现一个新的动画序列容器SCRAnimationChain，来解决上面所说的问题。为什么前缀是SCR呢？因为是我名字（尚传人）拼音首字母的大写，而且根据《Effective Objective-C》的建议，由于苹果宣称保留使用所有两字母前缀的权利，为了避免以后可能跟系统库的前缀重名，所以最好使用三个字母前缀。

思路如下：

1. 由于动画的动作被封装在块中，所以将动画抽象成Action类（封装Block），并且包含delay、duration、options等属性，用来保持最大的灵活性。
2. 参考GCD的思路，实现Sequence跟Concurrent两种Container类，分别用来针对串行动画跟并行动画的需求。
3. 由于Container与Action，类似文件目录与文件的关系，所以用组合模式抽象出一个接口类（Protocol）来处理他们之间的关系。这样的好处就是只要实现了接口类，不管具体是什么都可以放置到Container中，灵活性跟可扩展性都很强。
  
## 3. 实现

项目源码在[这里](http://github.com/joeshang/SCRAnimationChain)。

### SCRAnimationActionProtocol

接口类，相当于组合模式中的Component，在Objective-C中用协议描述。

- `(void)runWithCompletion:(SCRAnimationCompletionBlock)completion`：动画的核心就是运行，而为了能够形成链，所以增加了completion参数用来形成链。
- `(NSTimeInterval)workTime`：在并行动画中需要找到最终结束时间最长的那个动画，这样才能将并行动画
- `(void)addAction:(id<SCRAnimationActionProtocol>)action`

### SCRAnimationAction

封装具体的动画，相当于组合模式中的Leaf，里面的属性基本上是为了在调用`animateWithDuration:delay:options:animations:completion:`的时候用。

这里主要说明下`prepare`属性的用途。由于动画形成了序列，某个动画在执行时有可能前面的动画执行的一些操作会影响到当前动画，因此需要在执行此动画之前提供一个接口供其配置各种信息。

例如，在使用AutoLayout之后不能直接frame了，而是要使用contraint，在动画的块中调用contraint的superview（注意不能是contraint依附的view）的`layoutIfNeeded`。在动画序列开始前更新contraint的话，有可能前面的动画提前调用了`layoutIfNeeded`，导致动画出问题。（在Demo的ViewController中，你可以试试将那段在prepare块中的代码移到块之外，看会出现什么情况）

### SCRAnimationContainer

由于SCRAnimationSequence跟SCRAnimationConcurrence内部都有一个NSMutableArray来保存Action，所以将重复的代码抽出来，形成两者的基类。总之，DRY（Don't Repeat Yourself）！

### SCRAnimationSequence

串行动画的容器。利用递归的方式实现串行执行：在`runWithCompletion:`中每次从内部数组中取出一个项（由于是`id<SCRAnimationActionProtocol>`类型，所以可以是SCRAnimationAction，SCRAnimationSequence，SCRAnimationConcurrence或者实现SCRAnimationActionProtocol协议的任意类，这就是基于接口编程跟组合模式带来的灵活性），对其调用`runWithCompletion:`，在completion块中再次调用本身来实现递归。

### SCRAnimationConcurrence

并行动画的容器。直接用for循环，对内部数组的每一个Action调用`runWithWithCompletion:`，找出并行动画中workTime（delay + duration）最大的那个动画，将SCRAnimationConcurrence的completion块交给这个动画，确保completion能够得到执行。
