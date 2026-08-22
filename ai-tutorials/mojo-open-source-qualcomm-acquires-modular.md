---
title: "Mojo 开源那天，它的作者已属于高通"
slug: mojo-open-source-qualcomm-acquires-modular
summary: "Mojo 1.0 完全开源，编译器全部公开。但在这之前两个月，写它的人已经连同 150 人团队一起，被高通以近 40 亿美元买走了。"
category: ai-tutorials
tags: [Mojo, Modular, Qualcomm, CUDA, Chris Lattner, 编译器, AI 芯片]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: mojo-open-source-qualcomm-acquires-modular-en
---

# Mojo 开源那天，它的作者已属于高通

> 素材来自 Modular 官方博客（ModCon 2026 公告、Mojo 开源声明、Modular × Qualcomm 技术博客）与 WIRED 2026 年 6 月 24 日的收购报道。引语均为原文。

---

2026 年 8 月 18 日，Modular 在 ModCon 上宣布：Mojo 1.0 完全开源，Apache 2.0 协议，编译器和全部工具链公开在 GitHub 上。

对一门以「取代 CUDA」为使命的语言来说，这是它能做的最彻底的一件事。

但那份公告的页脚写着一行小字：**Copyright © 2026 Modular Inc, A Qualcomm Company.**

六月，高通宣布以近 40 亿美元收购 Modular，交易在 ModCon 前完成交割。等到 Mojo 的编译器源码真正公开的那天，写它的人已经在高通的组织架构图里了。

Chris Lattner 用了四年时间，想以一种「结构性」的方式打破一家芯片公司对 AI 软件栈的垄断。

结果是另一家芯片公司把他买了下来。

下面是这件事里，我认为真正值得琢磨的 10 个点。

---

## 1. 40 亿美元买的不是编译器，是「不用重写」这四个字

> "We believe the future belongs to developer-friendly, horizontal platforms that can run across diverse compute environments and give customers real choice in how and where they deploy AI."
>
> —— Cristiano Amon，高通总裁兼 CEO
>
> 译：我们相信未来属于对开发者友好的横向平台，它能跨越各种计算环境运行，让客户在如何部署、在哪部署 AI 上拥有真正的选择权。

先看价格。高通将发行最多 1920 万股普通股，按公告前收盘价算接近 40 亿美元。而九个月前，Modular 刚以 16 亿美元估值融了 2.5 亿。九个月，2.5 倍。

团队规模是 2 位联合创始人加上约 150 名员工，全部加入高通。按人头摊，每人约 2600 万美元。

这个价格显然不是在给一个编译器估值。高通买的是一句承诺：客户换硬件的时候，代码不用重写。

Modular 技术博客里那句话说得最直白：每有一款新 AI 加速器上市，开发者都要问「我们的软件栈要重写多少」，而 Modular 的答案是——**一行都不用**。

**My take：** 高通买的是时间，不是技术。芯片公司自建一套能跟 CUDA 抗衡的软件栈，历史给出的答案是五到十年，而且大概率失败——AMD 的 ROCm 已经证明了这条路多难走。40 亿美元换掉这五年，在一个 AI 推理硬件三年一换代的市场里，是笔便宜账。真正贵的从来不是芯片，是让人愿意为你的芯片写代码。

---

## 2. 「结构性」这个词，最后反噬了提出它的人

> At the time, Lattner said he believed that he and Davis were tackling a software problem that had to be solved outside of a Big Tech environment because it was "structural." Ultimately, the structure of Qualcomm won out.
>
> —— WIRED，2026 年 6 月 24 日
>
> 译：当时 Lattner 说，他相信他和 Davis 要解决的软件问题必须在大公司之外解决，因为这是个「结构性」问题。最终，赢的是高通的结构。

这是整篇 WIRED 报道里最锋利的一句，也是整件事的题眼。

Lattner 的原始判断是：AI 软件栈的碎片化不是技术问题，是结构问题——每家芯片公司都有动机把软件绑死在自家硬件上，所以在任何一家大公司内部都解不开。要解，必须站在所有硬件之外。

这个判断本身没错。错的是，他低估了「站在所有硬件之外」这个位置的经济学。

**My take：** 中立的软件层有一个先天缺陷——它对谁都有用，因此谁都不肯为它付真正的钱。芯片厂愿意为「能让我的芯片卖出去的软件」付钱，不愿为「能让我竞争对手的芯片也卖出去的软件」付钱。想解决结构性问题，就需要结构性的资源；而在这个行业里，结构性资源只有芯片公司手里有。这不是 Modular 一家的困境，是所有中立基础设施创业公司的宿命。

---

## 3. 他从来没有真正离开过硬件公司

> "What makes this team truly exceptional is the complementary partnership between Chris and Tim. Chris is an N-of-1 human, in that he's bold, visionary, and technically uncompromising."
>
> —— Dave Munichiello，GV 管理合伙人、Modular 早期投资人
>
> 译：这个团队真正卓越之处在于 Chris 和 Tim 之间的互补。Chris 是那种世上只此一个的人——他大胆、有远见、在技术上从不妥协。

把 Lattner 的履历摊开看，会发现一条别的线索。

LLVM 起于伊利诺伊大学的硕士论文，2005 年被苹果买下时才真正工程化。Clang、Swift、Xcode 工具链，全部诞生在苹果内部，他在那里待了 12 年。MLIR 诞生在谷歌，服务于 TPU。中间他去了特斯拉负责 Autopilot 软件——那段只持续了不到五个月，后来这个位置由 Andrej Karpathy 接手。再之后是 SiFive，一家 RISC-V 芯片公司。

现在是高通。

**My take：** 「开放基础设施」的历史，本质上是一部「哪家有钱的公司愿意为它买单」的历史。LLVM 之所以能成为今天的行业底座，不是因为它开源，是因为苹果连续十几年养着它。Lattner 最了不起的能力，从来不只是设计编译器，而是让一家巨头相信「养一套开放基础设施对我有利」。他离开大公司创业，某种意义上是想跳过这一步。绕了四年，又回来了。

顺带一提：Mojo 编译到高通的 Hexagon LLVM NPU 后端——把他送进高通的那座桥，是他自己二十多年前造的。

---

## 4. 编译器开源了，但暂时不收你的 PR

> "One learning (particularly in today's era of AI coding) is that we need to be deliberate about how we handle contributions. As such, we aren't ready to take contributions to the compiler and tooling. We aim to accept contributions to the compiler and tooling by the end of this year."
>
> —— Modular，Mojo 开源公告
>
> 译：我们的一个体会是（尤其在今天这个 AI 写代码的时代），必须谨慎处理贡献。因此我们还没准备好接受对编译器和工具链的贡献。我们的目标是在今年年底前开放这部分贡献。

源码全部公开，Apache 2.0 加 LLVM 例外条款，你可以读、可以改、可以 fork、可以拿去商用。

但你暂时不能往主干里提交代码。

Modular 给的理由是 AI 写代码的时代需要更谨慎的贡献治理——这个理由站得住脚，任何维护过热门开源仓库的人都懂那种被 AI 生成的 PR 淹没的感觉。

**My take：** 开源和共治是两件事，混为一谈是外行。Apache 2.0 给你的核心权利是**退出权**（不满意就 fork），不是**话语权**（决定语言往哪走）。这恰恰是 Lattner 一贯的语言设计哲学，公告里也写明了：语言的「灵魂」来自小而紧密的设计团队，不是委员会。Swift 早期也是这么走的。

所以别把「Mojo 开源了」理解成「Mojo 由社区共同决定了」。前者是事实，后者不是。真正值得看的信号是年底那个承诺兑不兑现，以及第一个来自社区的编译器 PR 是否被合并。

---

## 5. 真正的资产不是 Mojo，是 MAX——而 MAX 没有开源

> "The MAX license no longer contains device usage restrictions, and MAX will be source-available with an open alliance program, so the broader ecosystem can build the platform with us."
>
> —— Modular，ModCon 2026 公告
>
> 译：MAX 的许可证不再包含设备使用限制，MAX 将以「源码可见」的方式发布并配套开放联盟计划，让更广泛的生态与我们共建平台。

这一段容易被 Mojo 的头条盖过去，但它才是商业上更关键的部分。

Mojo 是**开源**（open source，Apache 2.0）。MAX 是**源码可见**（source-available）。这两个词在英文语境里是明确区分的：前者你能自由使用和分发，后者你能看见代码，但用途受许可证约束。

而 MAX 才是真正在赚钱的东西——图编译器、运行时、模型服务框架、跨硬件后端的整套推理栈。Modular Cloud 上跑的是它，MiniMax 每分钟数十亿 token 的生产流量跑的也是它。

**My take：** 语言开源换生态，平台闭源换收入，这是教科书级别的护城河布局，从 Java 到 .NET 到 Swift 都是同一套。这里没什么可指责的，但需要看清：Mojo 开源这件事的商业成本，比它的公关收益低得多。

判断 Modular 在高通旗下诚意的指标，不是 Mojo 现在的 license，是 MAX 一年后的 license。「source-available」和「开放联盟计划」这两个词，中间有很大的解释空间。

---

## 6. 「我们不是在抄 CUDA」

> "We are not copying CUDA; we are building a custom path into a portable stack."
>
> —— Modular × Qualcomm 技术博客
>
> 译：我们不是在复制 CUDA，我们是在为一个可移植的软件栈修一条定制通道。

这句话值得所有做「CUDA 替代」的团队抄在墙上。

配套的数字：Modular 和高通的工程师把 Cloud AI 100 Ultra 接进 MAX 和 Mojo，GPT-2 端到端跑通后第一版比基线**慢 1.6 倍**，三周优化后达到**基线的 6.7 倍**。Gemma 4 31B 从第一个 kernel 到线上服务端点，**不到六个月**。整个流程被拆成四个阶段：先让 Mojo 编译器在设备上跑起来，再用 GPT-2 当「Hello World」验证全链路，然后是 Gemma 4 31B 四卡张量并行上线，最后新模型和新特性从「几周」压到「几天」。

**My take：** 过去十年，试图挑战 CUDA 的方案基本都选了同一条路——做 API 兼容层，让 CUDA 代码原样跑在别人的硬件上。这条路的结构性问题是：你永远在追一个移动的靶子，而且靶子的所有者掌握着改变靶子的权力。

Modular 换了打法：不在 CUDA 那一层竞争，而是把抽象层级往上抬，让开发者写的是模型定义和 Mojo kernel，硬件差异吃在编译器里。这是唯一有机会赢的姿势——不是模仿垄断者的接口，是让接口这一层变得不重要。

至于那些数字，6.7 倍不是最重要的。最重要的是「三周」和「不到六个月」。**CUDA 真正的护城河从来不是性能，是迁移成本。**这次移植真正证明的，是迁移成本可以被压到什么量级。

---

## 7. NPU 不是 GPU，这次移植的含金量在这里

技术博客里有一段容易被跳过的硬细节，但它决定了这次移植值不值钱。

Qualcomm Cloud AI 100 Ultra 是一张卡上 4 个 SoC，每个 32GB 内存，每个 SoC 由 16 个 NSP 组成。它和 GPU 的差别不是量级上的，是范式上的：

- 它是 **SIMD 而不是 SIMT**，kernel 编程模型完全不同；
- **没有统一内存**，片上 VTCM 和片外 DDR 之间的数据搬运必须用 DMA 显式安排，不能指望缓存层级帮你藏延迟；
- 矩阵单元 HMX 要求数据是特定的 crouton 排布，转换有成本——所以 batch size 为 1 的 decode 步骤反而绕开 HMX，走向量单元 HVX 的 GEMV kernel，转换费不划算就不付；
- Gemma 4 31B 的 fp16 权重约 60GB，单个 SoC 装不下，必须四卡张量并行。

这是第一个进入 Modular 栈的 ASIC（NPU）。

**My take：** 可移植性这个词被滥用得厉害。从 NVIDIA 移植到 AMD 不算真考验——两者都是 SIMT、都有统一内存、编程模型同构，本质是方言之间的翻译。真正的考验是移植到一个执行模型完全不同的 ASIC 上，并且不 fork 模型定义、不 fork 图编译器、不 fork 服务层。

Modular 说所有硬件目标都在同一个仓库里、共享绝大部分代码。如果这是真的，那它证明的东西比 6.7 倍性能大得多：**AI 软件栈的硬件依赖，是可以被隔离在一层薄薄的抽象里的。**这正是 NVIDIA 最不希望被证明的事。

---

## 8. 高通押的不是芯片，是「横向平台」这个位置

把高通近两年的动作连起来看，收购 Modular 一点都不突兀。

去年收购 Ventana Micro Systems，一家做 RISC-V 服务器 CPU 的公司。同时在做数据中心定制 ASIC 设计，据报道字节跳动是早期客户。Amon 说公司手上有 **40 种**面向 AI 设备的芯片设计——智能眼镜、首饰、耳塞、胸针、手表。数据中心这边是 Cloud AI 100 系列和 Dragonfly AI 200/250/300。

现在再加上一整套软件栈。

**My take：** 高通的处境和当年的 ARM 有点像：在正面战场上不可能打赢 NVIDIA——制程、封装、HBM 供应、CUDA 生态，每一项都是十年积累。所以唯一的选择是改变竞争维度。

而软件是唯一能改变维度的杠杆。如果「写一次，跑在任何加速器上」成立，硬件采购决策的依据就从「哪家的软件生态好」变成「哪家的每瓦每美元性能高」——那正是高通几十年来在移动端最擅长的战场。

这不是买了个工具，是想改写评分表。

---

## 9. 中立性承诺，和承诺的保质期

> "Modular Platform will continue supporting and optimizing for a broad range of hardware, including hardware that competes directly with Qualcomm Technologies' platforms. The opportunity in front of the ecosystem is much bigger than any single vendor's roadmap, and a foundation only works if everyone can stand on it."
>
> —— Modular，ModCon 2026 公告
>
> 译：Modular Platform 将继续支持并优化广泛的硬件，包括与高通平台直接竞争的硬件。整个生态面前的机会远大于任何单一厂商的路线图，而一个地基只有在所有人都能站上去的时候才成立。

这段话写得很好，也确实是必须写的——不写，AWS、谷歌、d-Matrix 明天就该重新评估合作了。

目前的证据也不算差：Modular Platform 已经支持 AWS Trainium、谷歌 TPU、NVIDIA 和 AMD 的 GPU。谷歌 TPU 的支持甚至是外部公司 HTEC 的工程师自己接进去的，Modular 只做辅助——这是对「生态能自行扩展」最有力的证明。

**My take：** 收购当天，这句话百分之百是真的。问题不在诚意，在排期。

三年后当 Dragonfly AI 300 的支持和某个竞争对手新芯片的支持撞在同一个季度，谁排前面？没有哪家公司的 CEO 会在这种问题上选竞争对手。这不需要阴谋，只需要资源有限、KPI 是高通定的。

所以别看博客，看排期。**具体的检验点：Trainium、TPU、d-Matrix 的生产级支持是否和 Dragonfly 系列同步推进。**如果一年后其他硬件的支持还停在「未来几个月内投入生产」，那句承诺就已经作废了——不是被撕毁的，是被优先级慢慢饿死的。

---

## 10. 开发者现在到底该不该学 Mojo

把三年的路径连起来看，Modular 的开源节奏其实非常克制：2024 年开标准库，2025 年开 MAX kernels（几十万行），2026 年开整个编译器。每一步都是在生态压力和商业保留之间找平衡。

再加上这次 ModCon 的其他消息：Mojo 原生支持 Windows（和微软 Windows 团队合作，此前只能走 WSL）；Modular Cloud 正式公开可用，此前它以 ModelRun 的名字在 OpenRouter 上悄悄跑了几个月的生产流量，延迟和吞吐长期排在平台前列。

这些都不是 PPT，是已经在跑的东西。

**My take：** 这个问题要拆成两层答，因为这次收购把两层的风险推向了相反的方向。

**押注语言的风险变小了。**Mojo 1.0 给了源码稳定性保证，Apache 2.0 加 LLVM 例外给了最彻底的退出权，编译器全部开源意味着最坏情况下社区可以 fork。语言层面，你今天写的代码不会因为公司易主而废掉——这是开源在这个时间点上真正的意义，也可能正是它选在收购完成后才开源的原因之一。

**押注平台的风险变大了。**MAX 和 Modular Cloud 现在归属于一家有明确硬件立场的公司。如果你的推理业务要长期跑在这套栈上，你需要一个 exit plan，需要盯着中立性承诺的兑现情况。

具体建议：如果你做的是高性能 kernel、异构计算、编译器方向，Mojo 现在值得投入——编译器全开源之后，它对这个领域的人已经从「一个产品」变成了「一份可以研究的实现」。如果你在为生产推理选平台，把它当成一个技术上非常出色、但归属已定的选项来评估，别当成中立基础设施。

---

## 最后

四年前 Lattner 下的赌注是：AI 不会永远只跑在一种芯片上，软件栈必须为异构硬件重新设计。

这个赌注赢了。异构计算确实来了，Trainium、TPU、NPU、各种 ASIC 都在往生产环境里挤，而它们确实需要一个共同的软件地基。

只是这个地基最后没能站在所有人之外——它站进了其中一家的资产负债表里。

CUDA 的垄断有没有被打破，现在下结论还太早。但可以确定的是，打破它的方式，不会是一家独立公司靠中立性赢下来。

那句话值得再读一遍：**Ultimately, the structure of Qualcomm won out.**
