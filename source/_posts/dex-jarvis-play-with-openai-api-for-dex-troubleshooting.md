---
title: 基于 OpenAI Assistant API 与现有 Metrics 工具自动定位和修复 IT 问题的尝试
date: 2023-12-13 22:28:29
categories: OpenAI, LLM
---

## 背景

对于现代公司而言，其使用的 IT 系统稳定性和性能极大的影响公司的效率。因此很多大型公司，尤其是跨国公司会维持一个比较大的 Support Team，有的需要保持 7 * 24 小时在线，有的要支持全球化，这对公司而言是一笔不小的开销。另外 Support Team 还要进行各种培训，同步处理各种情况的 FAQ 手册和 Troubleshooting 文档。

但对于员工而言，目前常见的 Ticket 系统体验并不算理想：开 Ticket -> 描述现象和当前环境 -> 论坛回帖式的对话 -> 不断询问更多环境细节 -> 终于有可疑点但已经没有故障现场的数据了，导致处理流程很长，效率不高，时间大量浪费在 Round-Trip 式获取细节和 Metrics 中。

公司出于成本考虑总想削减 Support Team，而削减 Support Team 会导致员工遇到问题时需要更长时间才能恢复，降低员工满意度和效率。公司和员工都很疼，那该如何解决呢？能否将过程自动化呢？可将处理 IT 问题的流程自动化不太好做，因为问题来源端是人，是自然语言，而自动化需要标准接口。

这个问题困扰我很久，直到 LLM 和 Function Calling 的出现。

## 自动化的挑战

将流程自动化，我认为有三点需要处理：
1. 如何理解自然语言？
	a. 如何将灵活的自然语言转化成结构化的调用？
	b. 如何对问题进行分类？
	c. 如何根据问题的分类触发不同动作来收集 Metrics?
2. 如何排查问题并自动修复?
	a. 书写自然语言的 Troubleshooting 文档要比写复杂的规则或代码容易多，门槛也低。
	b. 怎么处理多步骤和条件？Troubleshooting 文档中某些步骤要不要做依赖上一步的结果。
3. 如何改进自动化？
	a. 并不是所有的 Case 都能够被自动化处理的，如何在无法全自动化时仍能提高效率？
	b. 如何可以通过经验不断提升和改进？
	

## 现有方案的局限性

### ChatGPT：需要的是解决方案，不是一堆选项
![](/dex-jarvis-play-with-openai-api-for-dex-troubleshooting/17316037118914.jpg)

当通过 ChatGPT 询问类似“为什么我无法打开某个网页？”时，可以看到 ChatGPT 提供了很多个可能原因的选项，这反而对用户造成困扰，应该从哪里解决呢？另外当问一些和内部系统紧密相关的问题时，ChatGPT 的表现也不好，因为它没有训练过这种数据。

总之，ChatGPT 虽然解决了理解自然语言的问题，但是体验一般，主要原因是：
1. 缺少实时的 Metrics 数据：提供给 ChatGPT 做判断的“现场”数据太少，导致 LLM 无法在较少 Context 下给出精确答案，只能列出一堆选项。
2. 缺少领域知识：没有训练过。

### 文档 QA 系统
![](/dex-jarvis-play-with-openai-api-for-dex-troubleshooting/17316041387314.jpg)

为了解决领域知识的问题，一种方案是将领域知识进行 Embedding 后放入向量数据库，当用户询问相关问题时，将问题和向量数据库中查询到的相关知识一起传入 LLM 模型，从而得到更符合的答案。这种文档 QA 系统在很多场景下工作的很好，但是由于缺少实时 Metrics 数据，还是无法准确定位到问题。

## OpenAI Assistant API + 实时 Metrics 数据

![](/dex-jarvis-play-with-openai-api-for-dex-troubleshooting/17316596091297.jpg)

最近 OpenAI 在其 Dev Day 上推出了 [Assistant API](https://openai.com/index/new-models-and-developer-products-announced-at-devday/)，可以将 Prompt、Function Call、File Retrieval 和模型结合起来，提供一种智能 Assistant 的体验。在仔细阅读了 Assistant API 相关文档后，发现将现有的 Metrics Agent 系统与 Assistant API 结合起来，能够为 IT 问题的自动定位和解决提供一套方案出来：
* 利用 Function Call 和 Prompt 将自然语言转成结构化的请求。
* 利用 Retrieval 实现领域知识的学习，包括知识库和故障排查手册等。
* 利用 Metrics Agent 根据用户提出的问题类型，收集相关的 Metrics 数据。
* 利用 LLM 模型，结合领域知识，分析 Metrics 数据，提供给用户准确的解决方案，Provide Solution Not Options。

### 理解自然语言

![](/dex-jarvis-play-with-openai-api-for-dex-troubleshooting/17316599095964.jpg)

当用户以自然语言询问自己遇到的问题时，在非人类介入的情况下，如何理解问题，如何对问题进行分类是个难题。而借助 Function Call 的能力，在定义如下的 Function Call 配置时，LLM 会将用户输入的自然语言转换成例如 `troubleshoot_network_issue(url, type)` 的结构化函数调用，我们就可以在函数中，根据参数传入的 url 与 type，用代码触发 Metrics Agent 进行相关的 Probe，收集 Troubleshooting 所需要的 Metrics 数据和 Context。

![](/dex-jarvis-play-with-openai-api-for-dex-troubleshooting/17325102234973.jpg)


### 结合实时 Metrics 数据和领域知识定位问题
![](/dex-jarvis-play-with-openai-api-for-dex-troubleshooting/17319197448600.jpg)

通过 Retrieval 能力把 IT Admin 日常使用的知识库和 Troubleshooting 文档上传到 Assistant 中进行向量化和学习，当 Metrics Agent 收集到相关的 Metrics 数据之后，将 Metrics 通过 Function Call 进行返回，LLM 将结合实时收集的 Metrics 数据和向量化后的领域知识，以自然语言的形式返回给用户解决方案。

同时可以通过 Function Call 提供给用户更方便的问题修复能力。例如如果 LLM 根据实时数据和领域知识判断出来是因为 VPN 没有打开，解决方法是尝试打开 VPN。可以在 Assistant 中定义 `toggle_vpn` 的 Function Call，用户可以用“帮我打开 VPN”这类自然语言触发该 Function Call，然后 Function Call 中的代码自动帮用户打开 VPN。甚至在配置好多个 Function Call 时，LLM 给出解决方案可以直接配上“需要我帮你打开 VPN 吗？”之类的回答。用户只需要回答“是”，LLM 配合上下文记忆也能够自动触发打开 VPN。

![](/dex-jarvis-play-with-openai-api-for-dex-troubleshooting/17325130200356.jpg)

### 和现有系统集成

![](/dex-jarvis-play-with-openai-api-for-dex-troubleshooting/17319197960624.jpg)

平时在值班和排查故障过程中，一个很大的痛点是无法及时获取故障现场的各类数据，包括通过和用户沟通获取到的信息。通过 LLM，即使无法自动解决用户故障，也可以在故障现场及时收集数据，同用户沟通，然后接入现有的 Support 系统自动开 Ticket，将对应的 Metrics、Log、Context 作为附件添加到 Ticket 中，提高 IT Admin 在排查故障时的效率，也省去用户繁琐的开 Ticket 的过程。

![](/dex-jarvis-play-with-openai-api-for-dex-troubleshooting/17319198104785.jpg)

当 IT Admin 有新的领域知识需要更新时（包括新的产品，新的 Metrics 数据，新的故障排查案例和步骤等），可以将其更新到现有的且人类可读的知识库与 Troubleshooting 文档中。这些领域知识一方面可以作为 IT Admin 团队的内部培训资料，对新入职员工进行培训或者团队内部知识同步，同时可以更新给 LLM 学习，提高能够自动化处理故障的覆盖率。这样一份资料可以让人和机器都进步，维护起来也容易，因为是人类可读的文本，不是代码或正则表达式。

## 实现

![](/dex-jarvis-play-with-openai-api-for-dex-troubleshooting/17316606509767.jpg)

在实现这个想法的过程中，给项目起名为 DEX Jarvis，希望能像钢铁侠中的人工智能助理 Jarvis 一样在 DEX 领域提供自动解决问题的服务。

整个项目的架构为：
* 用户侧：
	* Jarvis Chat：对话时交互界面，用户可以输入问题，以自然语言对话的形式得到解决方案。
	* Jarvis Voice：可以进行语音交互，用户语音进行 Voice Recognition 后将文本输入，而回答文本用 TTS 进行输出。
* LLM：基于 OpenAI Assistant API 实现的 Jarvis Agent。
	* Instruction：对 Assistant 进行 Prompt 配置。
	* Function Call：配置触发 Metrics Agent 收集数据和本地 IT 操作的结构化调用。
	* File Retrieval：从 Jarvis Knowledge Base 中获取和向量化领域知识。
	* Fine-Tune：从 Data Server 中构造各种 Case 标注后进行 Fine-Tune。
* Jarvis Scheduler：负责从 LLM 收到 Function Call 触发的 Task 并调度到用户设备上的 Metrics Agent，并在收集到 Metrics 数据后将结果返回给 LLM。与现有的 Support 系统对接，可以将 Metrics 数据和 Context 自动开 Ticket 后配上。
* Jarvis Knowledge Base：人类可读的知识库，里面包括相关领域知识和排查步骤，用于人类学习和 LLM 向量化。
* Metrics Agent：负责从用户设备上收集各类性能数据，并将数据上传到 Data Server。也可以在收到指令后进行主动探测或者自动操作用户设备上某些功能。
* Data Server：负责存储、聚合、监控、分析用户设备上的性能数据。
	

## 案例

下面是使用 Demo 演示的一个案例：

![](/dex-jarvis-play-with-openai-api-for-dex-troubleshooting/17319197244658.jpg)
在 Dex Jarvis 的对话界面上问为什么打不开某个 URL。此时 Jarvis 会触发 `troubleshoot_network_issue` 的 Function Call，从用户的自然语言输入中解析出 URL 作为 url 参数，type 参数为 can't open 类型。

此时 Metrics Agent 会根据 Function Call 和其参数，调用代码收集性能数据，例如：
* 检查本地网络连接是否正常，如果是 WiFi，检查 WiFi 信号强度。
* 检查 VPN 是否安装，是否打开。
* 检查该 URL 的 DNS 解析是否正常，是否为内网 URL。
* 对该 URL 进行 ping 和 traceroute 操作，收集网络时延。
* 模拟对该 URL 进行 HTTPS 请求，并收集每个阶段的用时或者错误。

![](/dex-jarvis-play-with-openai-api-for-dex-troubleshooting/17319197583964.jpg)
Metrics Agent 将上述整个性能数据收集完成后返回给 LLM，结合已经 Retrieval 过的故障排查文档，LLM 分析出来由于 URL 是内网 URL，而 VPN 并没有打开，所以无法打开的原因是 VPN 没有打开，并且检查出 WiFi 信号较弱，也可能导致访问慢的问题。

![](/dex-jarvis-play-with-openai-api-for-dex-troubleshooting/17319197726945.jpg)

同时 LLM 会问你是否需要帮忙打开 VPN，当回答是后，会触发 `toggle_vpn` 的 Function Call，可以在用户设备上自动帮用户打开 GlobalProtect 之类的 VPN。

## 总结

这个项目并没有什么很深的技术，主要是基于 OpenAI Assistant API 实现的一次探索，当了一次 API Boy，也没有考虑落地时 API 的使用价格等。但是在实现过程中，Assistant API 的 Function Call 与 File Retrieval 让我感到惊艳：将自然语言转换成结构化函数的准确，对人类可读的排查文档的理解，以及甚至向量化文档可以结合多个 Function Call 进行组合处理，达到了超出预期的效果。

探索这个项目，除了是看了 OpenAI DevDay 之后想把玩一下新出的技术外，也是对以前在猿辅导直播教室遇到的痛点念念不忘。教育领域的直播课，对质量要求较高，一旦直播体验出现问题，将影响整个教学体验。当时每周都会有很多用户报障，报告各式各样的问题，对于开发团队，除了业务功能开发外，会花费大量时间处理这些用户报障问题。很多时候要么问题是重复的，要么是环境问题，要么时间都浪费在获取当时现场或和用户沟通故障表现上，一直在想有没有什么办法能够提高报障处理效率，甚至自动化的处理。Assistant API 让我看到了这种可能性，相信随着技术的进步，后面还会有更多有意思的东西出现，真正能解决问题，提高效率，Stay Hungry Stay Foolish。