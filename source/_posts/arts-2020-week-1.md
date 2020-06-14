---
layout: 
title: ARTS 2020 Week 1：05.26-05.31
date: 2020-05-31 17:06:31
categories: ARTS
tags:
---

## Algorithm
[Invert Binary Tree - LeetCode](https://leetcode.com/problems/invert-binary-tree/)

决定从递归思想 + 树开始练习，周末家里有突发情况，所以选了一道简单+“有名”的翻转二叉树。树的结构由于自带子节点，所以很适合递归思想，对于这道题，翻转二叉树就是递归交换子树，递归起来可以有两种思路：

1. 每次交换的是左右子树，至于左右子树的结果，调用 invertTree 获取
```c++
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if (root == nullptr) {
            return nullptr;
        }
        TreeNode *tmp = invertTree(root->left);
        root->left = invertTree(root->right); 
        root->right = tmp;
        return root; 
    }
};
```

2. 每次交换左右子节点，然后再递归调用左右子节点
```c++
class Solution {
public:
    void invertTree(TreeNode* root) {
        if (root == nullptr) {
            return nullptr;
        }
        TreeNode *tmp = root->left;
        root->left = root->right;
        root->right = tmp;
        invertTree(root->left);
        invertTree(root->right);
    }
};
```

## Review
[Why iOS Developers Feel Stuck In Their Careers & What To Do — Essential Developer](https://www.essentialdeveloper.com/articles/why-ios-developers-feel-stuck-in-their-careers-and-what-to-do)

对于目前阶段的我而言，一直处于焦虑之中，一方面随着工作年限变多，无论是自己还是“业界”，对于自己的要求变得更高，另一方面，iOS 或者客户端相对服务器端而言，离业务有点远，核心竞争力并不突出。看到这篇文章，没想到”浓眉大眼“ Work Life Balance 的国外 iOS 同行也会 Feel Stuck。。。

文章的核心观点和我的看法是：
* 不要过于急功近利，设置不切实际的目标。学习的过程是曲折向上的，需要花时间持续投入，不要期望有立竿见影的效果，先坚持一段时间再说（例如 ARTS 活动）。
* Feeling Stuck 的原因有时候和工作环境有关，有些事可以缓解，例如同优秀的人合作（remarkable people），例如团队中有 mentor 可以指导如何高效的写和维护高质量的代码等等。关于环境这块我是这样想的，虽然环境会影响人，但个人是可以潜移默化影响环境的，要做“催化剂”
* 将工作中遇到的挑战与技术成长结合起来。工作不仅仅是编程或者技术本身，即使是一个简单的需求，那如何在 Commit 拆分上更清楚，如果是重复劳动，能否有一些自动化的工具来提高效率？在遇到例如网络超时问题时，能否更深入的排查，使用 Charles + WireShark 工具，重新翻看 TCP/IP 等书籍查漏补缺？另外，软技能一样重要，沟通能力、领导力等，也可以提高。


## Tips
1. 当 UILabel 的 adjustsFontSizeToFitWidth 为 YES 时，UILabel 会根据内容多少来调整字体大小，但是此时 baseline 不变，会导致文字在 Y 轴不居中。解决办法是：将 baselineAdjustment 设置为 UIBaselineAdjustmentAlignCenters。
2. UIScrollView 的 directionalLockEnabled 能够锁死每次滑动只影响一个方向，但是当滑动是对角线的情况，就失效了，需要在 beganDragging 时记录初始 offset，并将 direction 初始化为 .none，在 DidScroll 时判断 direction 类型，并根据 vertical 或 horizontal 来设置 contentOffset，最后在 DidEndDecelerating 和 DidEndDragging（willDecelerate 为 false）时重置为 .none
3. Swift 会对其符号进行修饰（Name Mangling），具体原理见：[mikeash.com: Friday Q&A 2014-08-15: Swift Name Mangling](https://mikeash.com/pyblog/friday-qa-2014-08-15-swift-name-mangling.html)。在 Bugly 上，如果崩溃在 Swift 方法中，被修饰过的命名就很难读了。Xcode 提供了对 Swift 符号进行 demangling 的工具，在命令行输入 `xcrun swift-demangle`之后，将对应的 Swift 符号拷入，点回车，就能看到解析后的结果了

## Share
[CNAME 有什么用？](https://joeshang.github.io/2020/06/14/cname-explain/)


