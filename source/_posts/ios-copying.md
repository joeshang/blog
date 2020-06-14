---
title: iOS 中的 Copying
date: 2018-04-04 10:29:12
categories: iOS
tags:
---

Copying 在 iOS 中有很多概念，例如浅拷贝与深拷贝、copy 与 mutableCopy、NSCopying 协议，一直想彻底搞明白这些概念，刨根问底不搞懂不罢休嘛。于是搜 Google 看了一些博客，又去翻了 Apple 相关的文档，发现网上许多博客都理解错了，下面说说自己的理解。

## 浅拷贝与深拷贝

对于浅拷贝（Swallow Copy）与深拷贝（Deep Copy），经常看到这样的说法：`浅复制是指针拷贝，仅仅拷贝指向对象的指针；深复制是内容拷贝，会拷贝对象本身。` 这句话并没有说错，但需要注意的是**指针/内容拷贝针对的是谁**，无论浅拷贝还是深拷贝，被拷贝的对象都会被复制一份，有新的对象产生，而在复制对象的内容时，对于被拷贝对象中的指针类型的成员变量，浅拷贝只是复制指针，而深拷贝除了复制指针外，会复制指针指向的内容。下面我们以 Apple 官方文档中的图片进行说明：

![Object Copy](/images/blog/ios-object-copy.png)

对普通对象 ObjectA 进行 copy，无论浅拷贝还是深拷贝，都会复制出一个新的对象 ObjectB，只是浅拷贝时 ObjectA 与 ObjectB 中的 textColor 指针还指向同一个 NSColor 对象，而深拷贝时 ObjectA 和 ObjectB 中的 textColor 指针分别指向各自的 NSColor 对象（NSColor 对象被复制了一份）。

![Collection Copy](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Collections/Art/CopyingCollections_2x.png)

对集合对象  Array1 进行 copy，无论浅拷贝还是深拷贝，都会复制出一个新的对象 Array2，只是浅拷贝时 Array1 与 Array2 中各个元素的指针还指向同一个对象，而深拷贝时 Array1 和 Array2 中各个元素的指针分别指向各自的对象（对象被复制了一份）。

## Copy 与 MutableCopy

在说明 copy 与 mutableCopy 之前，我们思考一下：拷贝的目的是什么？在动态库加载时，只读的 TEXT 段是被所有使用动态库的程序共享的， 而可写的 DATA 段会使用 COW（Copy On Write）技术，当某个程序需要修改 DATA 段时会拷贝一份，供此程序专用。因此，拷贝的目的主要用于拷贝一份新的数据进行修改，而不会影响到原有的数据。如果不修改，拷贝就没有必要。

在 iOS 中，有一些系统类根据是否可变进行了区分，例如 NSString 与 NSMutableString，NSArray 与 NSMutableArray 等。为了在两者之间进行转换（我理解这是主要目的），NSObject 提供了 `copy` 与 `mutableCopy`  方法， `copy` 复制后对象是不可变对象，`mutableCopy` 复制后对象是可变对象。对象有不可变对象和可变对象，复制方法有 `copy` 和 `mutableCopy`，因此存在四种情况：

1. 不可变对象 `copy`：对象是不可变的，再复制出一份不可变对象没有意义，因此**根本没有发生任何拷贝**，对象只有一份。
2. 不可变对象 `mutableCopy`：可变对象的能够修改，原来的不可变对象不支持，因此需要复制出一个新对象，是**浅拷贝**。
3. 可变对象 `copy`：不可变对象不能修改，原来的可变对象不支持，因此需要复制出新对象，是**浅拷贝**。
4. 可变对象 `mutableCopy`：可变对象的修改不应该影响到原来的可变对象，因此需要复制出新对象，是**浅拷贝**。

### 如何进行深拷贝呢？

对于集合类型的对象，将 `initWithArray:copyItems:` 第二个参数设置成 YES 时，会对集合内每一个元素发送 `copyWithZone:` 消息，元素进行复制，但是对于元素中指针类型的成员变量，依然是浅拷贝，因此这种拷贝被称为单层深拷贝（one-level-deep copy）。

如果想进行完全的深拷贝，可以先通过 NSKeyedArchiver 将对象归档，再通过 NSKeyedUnarchiver 将对象解归档。由于在归档时，对象中每个成员变量都会收到 `encodeWithCoder:` 消息，相当于将对象所有的数据均序列化保存到磁盘上（可以看成换了种数据格式的拷贝），再通过 `initWithCoder:` 解归档时，就将拷贝过的数据经过转换后读取出来，深拷贝。

### NSCopying

如果自定义的类也想要支持 `copy` 和 `mutableCopy` 方法，就需要实现 `NSCopying` 和 `NSMutableCopying` 协议。在实现 `copyWithZone:` 方法时需要注意：
-  `copyWithZone:` 相当于新创建一个对象，并将当前对象的值复制到新创建的对象中。设置时应直接访问成员变量而不是通过属性访问。
- 直接从 NSObject 继承的类，应使用 `[[[self class] allocWithZone:zone] init]`，使得在创建新对象时能够使用正确的类。
- 父类中已经实现了 `copyWithZone:` 时，应先调用父类的方法，让父类创建对应的对象（self class 能保证创建对象是正确的），并拷贝父类中定义的成员变量。

```objc
- (id)copyWithZone:(NSZone *)zone {
    YourClass *object = [super copyWithZone:zone];
    _property = xxx;
    return object;
}
```

参考：
- [Object copying](https://developer.apple.com/library/content/documentation/General/Conceptual/DevPedia-CocoaCore/ObjectCopying.html)
- [Copying Collections](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Collections/Articles/Copying.html)
- [数据存储之归档解档 NSKeyedArchiver NSKeyedUnarchiver · pro648/tips Wiki · GitHub](https://github.com/pro648/tips/wiki/%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E4%B9%8B%E5%BD%92%E6%A1%A3%E8%A7%A3%E6%A1%A3-NSKeyedArchiver-NSKeyedUnarchiver)

