---
title: 如何在业务团队保持”技术前瞻性“？
date: 2021-03-29 01:00:51
categories: Thought
tags:
---

作为技术人员，一直很羡慕别人在基础设施领域做的一些很牛逼的工具和框架，虽然业务看起来就是在”搬砖“，但业务以及业务背后的服务才是一个公司的根本，这也是为什么有些公司技术并不牛逼，但发展却超出想象的原因。这并不代表对技术不重视，反而相反，将技术与业务结合起来，能够用合适的技术将业务支撑起来也是工程师的核心价值，毕竟工程师，就是“能将梦想照进现实的人”。

那作为一个业务团队，如何能够保持”技术前瞻性“，支撑业务的快速发展和迭代？所谓”前瞻性“，就是”晴天修屋顶“，听起来很好理解，但实际涉及到的问题有：
* 如何判断什么时候是晴天，即什么时候需要修？
* 应该修什么样的屋顶？
* 用什么工具和办法修？

我就从下面几个方面，谈谈我的思考

## 真正深入业务，了解业务全貌，跟进业务走向
在业务开发团队，开发同学经常有的迷茫和吐槽是：
* 感觉就是在搬砖，PM 给个需求就做，天天就在写需求，没意思，也没什么技术成长
* 这个需求感觉好傻啊，为什么要这样搞？这个需求又大又急，为什么这么急？代码越搞越脏

当然，不排除有运营或者 PM 提一些“拍脑袋”的需求，但这是业务开发团队相对难以改变的，要么换家公司（我感觉这方面都差不多吧？），要么拍回去，剩下能做的，就是从开发团队本身看看能做什么？

在业务开发团队，我认为非常重要的一点就是”真正深入业务“，对于开发同学，可能容易只看到技术，忽略业务本身。但在业务开发团队，技术是支撑业务的，只有深入了解业务，才能在业务角度做出”前瞻性“。

### 了解公司业务全貌
《Netflix 文化手册》中的文化准则 2 是”要培养基层员工的高层视角“。我以前也觉得自己就是”搬砖”的，战略、业务啥的都是大佬考虑，自己做好活就行了，后来发现不对：
* 公司需要的是聚焦，人多不一定力量大，人的力气往一处使才力量大，这也是 OKR 做聚焦的目的。当了解公司业务全貌后，能比较清楚知道自己做的工作是否和公司方向对齐，聚焦自己的工作。
* 团队变多后团队之间的交互反而容易出问题，因为没有人能总览全貌，当流程较长且对流程不熟悉时，整个项目容易出问题。而熟悉业务全貌后，能够发现团队间交互问题，提前暴露风险。

### 做需求时多问问背景
我觉得很多时候开发人员需求做得恶心，并不是因为难或者有技术挑战，反而是因为觉得没有意义或者不知道有什么意义，那这个时候需求背景就显得很重要了。
* 在看 PRD 时，我们经常忽略掉背景的 WHY，而只关注要做什么的 WHAT。当我们了解了公司业务，明白了需求背景，才能意识到这个需求有意义，做起来相对有动力一些，也更能从长期思维考虑，在实现时如何更全面。

### 跟进业务走向
有时候不明白为什么 PM 突然出了一个又急又大的活，这就需要我们多关注产品/UI OKR，多和 PM 聊聊天，提前探探他们之后想做哪方面的尝试，这样能够在技术上提前准备好。
* 例如要更活泼的交互，那就多调研动画框架等
* 例如要开更多的教室或课堂活动，那就做架构重构，提高配置灵活性等
* 例如要提高运营或者生产效率，那就分析流程，将流程平台化等

## 关注人员/组织变化，提高对接效率
在业务团队，除了关注业务上的演进方向外，还有一方面特别容易被忽略，就是：人员（组织）变化。

随着业务需求，公司或者团队可能会快速扩充某个团队或者组建新的团队，当团队人数急剧增加或者需要和新团队频繁对接时（很多情况不是技术团队扩张，而是非技术团队扩张），原来一些手动操作的工作就成为的效率瓶颈，也会让业务团队的开发感到”烦躁“，觉得自己每天都在做一些琐碎工作；同时团队间如果依赖过重，导致互相影响，在联调和问题排查时也非常难受。

那业务团队可以做的“前瞻性”工作就是提高效率。具体措施是：
* 将手动操作自动化：将之前手动跑的 SQL 或者操作脚本化，配置成 Job，自动定期跑 + 将结果推送给关心的人；将手动检查做成对非技术同学更友好的形式，搞成自动检查任务，每次自动检查。
* 将配置/管理平台化：以前人少的时候还可以自己手动改改配置，手动发布之类，随着人数增加，需要实现一个配置/管理的平台，交给使用方根据自己需求进行配置和管理，业务团队的开发同学就可以从琐碎配置中解放出来，主力维护平台。
* 隔离变化：当团队变多，需求的开发链条变长后，作为开发联调中的一环，需要隔离其他团队由于内部变化而导致的问题。

以上做法，除了提高效率外，还能应对团队规模变化，例如平台化了之后，由于对接的是平台，人数可以随意扩容，算法复杂度从  O(n) 降为 O(1)。

## 深入和扩充技术栈，真正用技术支撑业务
技术同学容易犯两种错误：
* 新技术是”银弹“：拿着锤子看啥都是钉子，为了上新技术而上新技术，或者为了造轮子而造轮子，而不考虑业务的落地场景，是否真正解决痛点。
* ”稳定压倒一切“：对新技术不敏感也没深入了解，要么对新技术嗤之以鼻，要么不知道有更合适的技术来解决业务问题，觉得保持现状挺好。

我个人觉得，第一个问题基础设施团队相对容易犯，第二个问题业务团队相对容易犯（只是个人看法，上面的问题我都犯过，现在也在不断自省，提醒自己不能迷恋”新技术“，也不能因为不了解某种技术就否定）。

因此对于业务团队，我认为能做的事情是：**深入和扩充**技术栈，**真正**用技术支撑业务。注意加粗的字，为什么要强调呢？

关于”深入和扩充“：在做业务开发时经常是够用就行，先尽快把业务实现，并不会深挖技术栈，也不想了解别的组是怎么实现的，但这样是不够的：
* 深入技术栈：如果要做到极致性能，就是需要深挖所在的技术栈，不深入了解，遇到疑难问题找不到思路和问题分析，遇到业务难点不知道怎么实现和优化，导致”技术深度不够”，无论对个人成长（例如出去面试），还是公司内成长（成为技术专家，在团队内营造“信任感”）都是不利的。
* 扩充技术栈：多扩充技术栈，一方面可以从别的技术栈学习好的思想和设计，看看能不能吸收自己所用的技术栈中；另一方面，多了解其他端的实现，并不是为了全栈一个人包圆了，而是在交流沟通上更加顺畅，也能站在更全面角度选择和评估技术方案。

关于”真正“：在深入和扩充技术栈后，开发同学手里有了“锤子”，特别想砸钉子，要忍住这种冲动。
* 把自己想象成一个工匠，深入/扩充技术栈只是往自己的工具集中增加了一种工具，工匠最主要的工作是做工艺品，要根据具体情况选用合适的工具，眼睛盯着的是工艺品，而不是工具。
* “真正”强调的是：在深入业务之后，针对业务中的痛点，看能否从新工具中找到一样合适的。

总之，通过深入和扩充技术栈，开发同学扩展了自己的工具集，可以针对业务中的痛点，用更合适的工具来解决问题，应对变化，达到“技术前瞻性”的目的。

## 解决问题时多思考长期改进
在遇到一些疑难问题时，多写总结，不能解决了就过去了，从长期角度可以看看有什么改进，这样也能保证“技术前瞻性”：
* 例如出现了内存泄露，把泄露处的代码改了是解决了问题，但从长期改进看，能不能搞一个内存泄露检测工具？
* 例如出现了性能卡顿，优化卡顿出代码也解决了问题，但从长期改进看，需要 APM 监控和自动化压力测试等

最后，我们再尝试回答最一开始的问题：
* 什么时候需要修？跟进业务走向 + 关注人员/组织变化 + 遇到问题时关注长期角度
* 应该修什么样的屋顶？真正深入业务，了解业务全貌 + 思考长期改进
* 用什么工具和办法修？深入和扩充技术栈，手动操作自动化，配置平台化，隔离变化