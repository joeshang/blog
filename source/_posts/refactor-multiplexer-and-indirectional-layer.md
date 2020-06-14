---
layout: title
title: 重构技巧：数据选择器与中间层
date: 2020-06-14 21:20:46
categories: Architecture
tags:
---

![](/images/blog/cs-of-luxun.png)

> Any problem  in computer science can be solved by anther layer of indirection  

在计算机领域有句名言：“计算机科学领域的任何问题都可以通过一个中间层来解决”，能找到很多例子：
* 虚拟内存： 为了更好的隔离和管理内存，在程序和物理内存之间增加虚拟的内存控件作为中间层。
* 操作系统：为了防止应用程序直接（随意）访问硬件，也为了降低使用硬件的复杂度，操作系统和驱动程序来作为中间层。
* JVM：Java 通过构造一个 JVM 虚拟机，隔离了不同平台的底层实现，使得 Java 的字节码可以多个平台上不加修改地运行。
* 其他还有很多，例如 TCP/IP、汇编等。

总之，中间层的核心思想，是通过层与层之间的接口，隔离两个层各自的细节和变化。这种间接性 Indirection 的思想除了在架构设计上得到应用，在一些需求变化导致的重构场景也比较适合，这类场景我称为“多路开关”，或者也可以叫“数据选择器”，具体请看下面。

## 数据选择器
在电子技术（特别是数字电路）中，数据选择器（Data Selector），或称多路复用器（multiplexer，简称：MUX），是一种可以从多个模拟或数字输入信号中选择一个信号进行输出的器件。

![](/images/blog/Multiplexer2.png)


在软件开发中，多个输入对应一个输出的场景也比较常见：列表页原来只使用一种类型的数据，在 TableView 的 DataSource 中都是直接使用对应的 Model，随着需求变化，多了一种类型数据，而 Cell 样式是相同的。

此时应该如何修改代码来比较稳的应对这样的变化？直接改吗？那原来使用数据的地方会多一堆 if/else 条件语句，难以维护不易读，如果再增加一条数据源，数据使用的地方还需要再次改动。。。

这类场景，和数字电路中的数据选择器类似，多路数据输入一路数据输出，由选择器负责切换数据输入。参考相同思路，也构造“数据选择器”：

1. 抽取中间层，构造“选择器”，重构原有代码接入选择器。抽象层可以用数据抽象，也可以用一个函数封装获取数据的方法，并将“选择器”相关的代码集中到一起，方便维护和处理。
2. 测试重构后的代码。由于第一步是通过抽取中间层构造“选择器”，数据的消费方不再是直接访问原来的数据，而是通过“选择器”获取数据，因此需要进行测试，保证没有重构出问题，那下一步接入新数据出现问题，就是“选择器”在选择时有问题。
3. 接入新的数据源。基于前面的重构，这一步的接入变得简单，专注于在“选择器”代码中根据业务需求选择走哪条数据源，数据消费部分的代码和逻辑完全不需要修改，同时选择器的选择逻辑也可以抽出来进行单测。

### 快速投票模块
直播教室内的快速投票原来是基于 Ballot 命令进行显示消失的，后来服务器换成 WidgetState 的方案，相当于一条新的数据源，并且服务器期望根据 Config 进行配置。

重构就是分三步走：
1. 将原来直接使用 Ballot 的地方抽到函数中，并将 Ballot 和 WidgetState 中共同使用的数据抽出来，使用无依赖的基础数据类型描述（例如 bool 或字典等），原来直接用 Ballot 的消费方，现在使用这些基础数据类型。
2. 测试第一步的重构。
3. 接入 WidgetState 的数据，在第一步的函数中，根据 Config 决定是从 Ballot 转还是从 WidgetState 转。

### 笔迹库重构
以前笔迹库只渲染笔迹，所有的操作栈（Undo、Redo、Lasso、ClearScreen）等都和笔迹 View 绑定较死，现在由于要接入图形，是一个新的数据源。

重构依然是分三步走：
1. 将操作栈从笔迹 View 中抽出，由一个专门的 Manager 来负责管理，并生成每一步的渲染数据，笔迹 View 就负责渲染笔迹相关的数据。
2. 测试第一步的重构。
3. 在 Manager 中接入图形数据，并实现一个图形 View 负责渲染图形相关的数据。

同时，为了保证上线的稳定性，需要有开关回退，那新旧两个 View 都在，根据开关进行区分，又是一个“数据选择器”。于是抽象一个 Protocol 做为中间层，外部使用是 UIView<Protocol>，由这个中间 View 根据开关切换。

最后来回顾一下所谓的“选择器“，这有什么新的东西吗？仔细看下“选择器”代码，其实就是由于多了一路数据而导致的变化，抽取中间层相当于将变化隔离开，嗯，最后还是“隔离变化”，万变不离其宗。