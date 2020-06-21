---
layout: title
title: 重构技巧：Parser 与多态
date: 2020-06-20 23:11:43
categories: Architecture
---
![](/images/blog/1A323420-7A1B-4934-8D05-32B2B53AFB1F.png)

对于 Parser，一般我们能想到的是同一个数据流，根据协议或者格式的要求进行区分，解析成不同含义的元素。这个解析过程一般存在着复杂的条件逻辑，用于匹配协议或者格式的要求。

抽象一下，可以将 Parser 看成有**复杂条件逻辑处理同一数据流**的场景。而复杂的条件逻辑是编程中最难理解的东西之一，复杂的 if/else 或者 switch/case 中包含了许多细节，容易引入 Bug ，也使得修改变得麻烦。

## 多态
在《重构 Refactoring》这本书中，针对上面的问题，有个技巧被称为“以多态取代条件表达式”（Replace Conditional with Polymorphism）

![](/images/blog/command-comic-1.png)

这个技巧的核心在于，将每一条分支逻辑隔离到一个类中，用多态来承载各个类型特有的行为，“上层”或者“业务层”不再关心每一条分支的具体细节，不再事事躬亲，只作分发（Dispatch）的工作。

## 案例
### JS 回调重构
最早的 WebViewController 在处理 JS 回调的方法是用一堆 if/else 语句：

```objc
- (void)jsCallback:(NSString *)name arguments:(NSDictionary *)arguments {
	if ([name isEqualToString:@“command1”]) {
		[self handleCommand1:name arguments:arguments];
	} else if ([name isEqualToString:@“command2”]) {
		[self handleCommand2:name arguments:arguments];
	} else if ([name isEqualToString:@“command3”]) {
		[self handleCommand3:name arguments:arguments];
	} else if ([name isEqualToString:@“command4”]) {
		[self handleCommand4:name arguments:arguments];
	}
}
```

这样写的问题是导致 WebViewController 越来越庞大，一堆业务逻辑耦合到 WebViewController 中（例如登录通知，语音跟读的回调等），维护性变差。另外，如果想配置 WebViewController 只支持某些或者不支持某些 JS 特定的回调的话，甚至根据页面 URL 进行动态调整，也不是很干净。于是趁着 UIWebView 升级 WKWebView，做了一次重构：基于命令模式，将 JS 回调的处理抽离到一个个 Handler 中，JS 回调的名称和参数也在 Handler 中维护，WebViewController 中不再含有任何与 WebView 无关的业务逻辑，当 WebView 触发了 JS 回调后，调用 Command Manager 这个 Invoker 去调用 Command。

```objc
- (void)registerCommands {
	[self.commandManager registerCommand:[Command1Handler new]];
	[self.commandManager registerCommand:[Command2Handler new]];
	[self.commandManager registerCommand:[Command3Handler new]];
	[self.commandManager registerCommand:[Command4Handler new]];
}

- (void)jsCallback:(NSString *)name arguments:(NSDictionary *)arguments {
	JSCommand *command = [JSCommand commandWithName:name arguments:arguments];
	[self.commandManager handleCommand:command];
}
```

### 图片标注操作栈
对于图片标注功能，支持笔迹、图片、文本、橡皮擦、套索等，同时有 Undo、Redo、ClearAll 等操作。

由于涉及到 Undo、Redo 操作，因此需要维护一个操作栈。基于此，需要将每种操作抽象成 Action，Action 中有 type 属性，用于描述 Action 的具体类型。同时定义 ActionManager 的类，负责维护操作栈，并基于操作栈实现 Undo、Redo 操作。

一开始的代码可能是这样的：
```objc
- (void)undo {
    if (self.currentIndex <= 0) {
        return;
    }
    self.currentIndex--;
    Action *action = self.actions[self.currentIndex];
    if (action.type == ActionTypeStroke) {
        // handle stroke
    } else if (action.type == ActionTypeLasso) {
        // handle lasso
    } ...
}
```

在 undo/redo 方法中，除了处理操作栈外，需要根据 Action 的不同，处理该类型 Action 在 undo 时应该做的事情。但回过头来看看 ActionManager 的职责，其没有必要了解 Action 的具体细节，因此，Action 应作为基类或者接口，定义 do/undo 两个方法，各个子类 Action 实现 do/undo 方法，分别在 ActionManager 在 redo/undo 中调用。这样修改之后，ActionManager 的逻辑变得清晰：

```objc
- (void)undo {
    if (self.currentIndex <= 0) {
        return;
    }
    self.currentIndex--;
    Action *action = self.actions[self.currentIndex];
    [action undo];
}
```

### SVG 解析库
最近看了下 [Skia 中 SVG Parser 的源码](https://chromium.googlesource.com/skia/+/chrome/m44/src/svg/parser)，虽然 Parser 中 switch 语句依然存在，但是 switch 中只是针对不同的标签（Path、Line、Rect、Circle 等）生成不同的 Element，至于如何解析 Element，由各个 Element 子类实现 Element 基类中定义的 translate 方法，负责解析出各自类型 Element 中的属性。

