---
title: "Mojo 开源了编译器，也悄悄改掉了「Python 超集」那句话"
slug: mojo-open-source-68000x-truth
summary: "2026-08-18，Modular 把 Mojo 编译器和全套工具链以 Apache 2.0（含 LLVM 例外）开源。但传播最广的两个卖点都需要更正：「比 Python 快 68000 倍」是 2023 年那组 Mandelbrot 博客的数字，基线是单线程纯 CPython 解释器循环，官方自己说过 35000x 变 68000x 只是因为换了台 88 核机器；而「Python 超集」这个定位，官方 roadmap 已经改成「可能会，也可能不会，不成也没关系」。另外还有一件公告里没强调的事：Modular 已经在 7 月 29 日被高通收购完成。本文核对开源的确切范围、benchmark 的成立条件、互操作的真实代价，以及现在该不该上手。"
category: developer
tags: [Mojo, Modular, Chris Lattner, 编程语言, 开源, Python, 性能优化, 高通, MLIR]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: mojo-open-source-68000x-truth-en
---

# Mojo 开源了编译器，也悄悄改掉了「Python 超集」那句话

8 月 18 日，Modular 在自家 ModCon 大会上宣布：Mojo 编译器和全部工具链开源，Apache 2.0 许可证加 LLVM 例外。

这条消息在中文圈的传播版本通常带两个卖点：**「Python 超集」**和**「比 Python 快 68000 倍」**。

问题在于，这两句话都不是这次公告说的。一句是三年前的旧数字，另一句已经被官方从路线图里改掉了。

把它们放回原始语境，你会得到一个比标题朴素、但更值得认真对待的判断。

---

## 一、开源的到底是哪一部分

先说清楚事实边界，因为「开源」这个词这次被用得比较宽。

Modular 的原文是：

> "We are happy to announce that the Mojo🔥 language is now fully open source under the Apache 2.0 license (with LLVM exceptions)! The source code for the Mojo compiler, tooling, and everything else you need to build the language are now available in our modular GitHub repository."

对应的此前状态，官方自己的描述很坦白：

> "For the last four years, Mojo has been developed with an open community, but a closed compiler."

所以这次真正新开的是**编译器本体和工具链**（LSP、调试器、格式化器、REPL），代码在 `modular/modular` 仓库的 `KGEN` 目录下。整条路径是分三步走完的：2024 年 3 月开标准库，2025 年的 25.3 版本开了 45 万行用 Mojo 写的 MAX kernel，这次补上最后一块。

许可证值得单独说一句：我核对了仓库 LICENSE 文件首行，确实是 **Apache License v2.0 with LLVM Exceptions**，没有 CLA 之类的商业附加条款。但 GitHub 侧边栏因为文件头部有自定义说明，识别成了 "Other"——**别照抄侧边栏**。官方选它的理由写得很直白："The Apache 2.0 license is the gold standard for programming languages and compilers."

有两个限定条件容易被跳过，但对判断「这算不算真开源」很关键：

**第一，编译器暂时不收外部贡献。** 官方原文：

> "we aren't ready to take contributions to the compiler and tooling. We aim to accept contributions to the compiler and tooling by the end of this year."

**第二，MAX 引擎不在这次的 Apache 范围内。** 仓库 README 明写 "MAX usage and distribution are licensed under the Modular Community License."。同日 ModCon 公告说 MAX 会走另一条路：去掉设备使用限制，然后变成 "**source-available** with an open alliance program"——注意官方对 MAX 用的词是 source-available，不是 open source。这个用词差别是有意的。

HN 上关于「这算不算开源」的那一支讨论，恰好把两种合理立场都摆出来了：

> Lichtso：「Technically source available now with the promise of accepting contributions (thus becoming fully open source) early next year... For many the closed source nature of the compiler was a knock-out criterion. We will see if Mojo can gain traction now or if it has missed its window of opportunity.」

> cube2222 反驳：「It's open source under the Apache 2 license, not source available. Accepting contributions is not required to be open source. SQLite doesn't accept contributions from random people either.」

cube2222 是对的：接受贡献从来不是开源定义的一部分，SQLite 就是现成的反例。但 Lichtso 那句「窗口期是不是已经错过了」，是这次公告下面重复出现最多的疑问。

顺带一个彩蛋：有人翻 `KGEN/docs/` 目录，发现编译器设计文档还带着当年的密级标记——HN 用户 ModernMech 的原话是 "'Modular Confidential (obviously), May 14, 2022' — lol, feels like espionage"。开源开得很彻底，连 2022 年的内部文档一起放出来了。

## 二、68000 倍：这个数字是 2023 年的，基线是单核解释器

**这次的开源公告和 1.0 公告里，都没有再提 68000x。** 它来自 2023 年官方的一个三篇博客系列，主题是 Mandelbrot 集（不是矩阵乘法）：

| 阶段 | 做了什么 | 倍数 |
|------|---------|------|
| Part 1 | Python 代码移植到 Mojo + 类型标注 + 代数化简 | 89x |
| Part 2 | SIMD 向量化 + 多核并行 | 26,000x |
| Part 3 | over-partitioning 做负载均衡 | 68,000x |

关键在于**基线是什么**：单线程的纯 CPython 解释器实现，不是 NumPy，更不是 C。

也就是说，68000x = 一个 88 核 CPU 上跑的、向量化的、编译执行的 Mojo，对比**一个单核解释执行的 Python for 循环**。

最能说明问题的是官方自己在 Part 3 里的解释——为什么倍数从发布时宣传的 35000x 变成了 68000x：

> "Why are we getting a 68,000x speedup instead of the advertised 35,000x speedup? In short, the benchmarking system is different. During launch, we evaluated on an AWS r7iz.metal-16xl 32-Core Intel Xeon Gold 6455B, but in this blog we evaluated on a GCP h3-standard-88 which uses an 88-Core Intel Xeon Platinum 8481C."

**换了台核更多的机器，倍数就涨了将近一倍。** 这句话本身是诚实的技术说明，但它同时也说明：这个数字衡量的很大一部分是「你租了多少核」，而不是「语言有多快」。

第三方那边：Julia 社区 2023 年 9 月做过对照实验，HN 上 sundarurfriend 的结论是 "the version of code in [1] is already a few times faster than the Mojo code - because that's pretty basic Julia code that anyone with a little Julia experience could write"——普通水平的 Julia 代码就快过 Mojo 官方文档里的实现。HN 当年还有一个专门的帖子叫《Is Mojo really 35000x faster than Python?》。

需要如实说明的是：**「换成 NumPy 或 C 基线是多少倍」，官方从来没给过数字，我也没找到权威的第三方单一结论。** 能确定的只有方向——基线一换，倍数会坍缩掉好几个数量级。

这不是 Mojo 独有的毛病，是整个性能宣传领域的通用套路。我们拆[「Codex 提速 232 倍」](/articles/codex-autoresearch-gpu-kernel-232x)时用的是同一个方法：**先问基线是什么，再看倍数是多少**。顺序反了，任何数字都能唬人。

## 三、「Python 超集」已经被官方改口了

2023 年 Mojo 发布时，最抓人的定位就是「Python 的超集」——你现有的 Python 代码原样能跑，然后逐步加类型标注换性能。

现在官方 roadmap 的 Phase 3 里写的是：

> "Mojo may or may not evolve into a full superset of Python, and it's okay if it doesn't."

「可能会，也可能不会，不成也没关系。」

而现行的官方 FAQ 里，superset 这个问答条目已经整条消失了。这是一次没有发公告的改口。HN 上 spprashant 的评价是 "They moved the goalposts."；pansa2 补了一句更实在的："It absolutely was [part of the appeal], but IMO was never going to happen... Python has a reputation as a simple language, but it really isn't."

那么今天的互操作到底是什么水平？

**能 import 任意 CPython 包，这是真的**，官方文档原文：

> "You can import existing Python modules and use them in a Mojo program. This is 100% compatible because we use the CPython runtime without modification for full compatibility with existing Python libraries."

但请注意后半句——**它用的是未经修改的 CPython 运行时**。意思是这些 Python 库仍然以 CPython 的速度运行，外加一层跨语言调用开销。HN 上一位 NuMojo 相关开发者 MohamedMabrouk 说得最清楚："you can but through a python interpreter in the mojo process so you get the same numpy speed with mojo<->python interop overhead."

**你 import 进来的 NumPy，不会因为跑在 Mojo 里就变快。** 加速只发生在你用 Mojo 重写的那部分。

至今仍不支持的 Python 核心特性，官方 roadmap 全部排在 Phase 3（也就是"以后再说"）：

- **class 和继承**——官方原话 "Eventually, we want Mojo to support the core dynamic features that make Python great, including untyped variables, classes, inheritance, etc."
- **无类型动态变量**——"For now, Mojo requires explicit `PythonObject` type annotations."
- **完整的 async 模型**——1.0 公告把它列在"未来能力"里："major capabilities ahead including a robust asynchronous programming model, pattern matching and unions"
- lambda 是 1.0 才加的（此前连 lambda 都没有）

所以更准确的描述是：**Mojo 是一门 Python 风格语法的系统级语言，能调用 Python，但不是 Python。** 它的真实血统在 HN 上被 totalperspectiv 一句话概括得挺好："It's got an ownership system adjacent to Rust, comptime similar to Zig, and a first class dependent type system."——Rust 的所有权、Zig 的编译期计算、外加依赖类型。这个组合和「Python 超集」是两码事。

## 四、公告里没强调的那件事：Modular 已经属于高通了

这是我认为整条新闻里最该被放进标题、却几乎没人提的背景。

- 2026-06-21 签署最终协议，06-24 官宣
- **2026-07-29 收购完成**，全股票，约 **39 亿美元**
- Chris Lattner 出任 Qualcomm Technologies 的 EVP, Advanced AI Software and Platforms
- 官网页脚现在署的是 "Modular Inc, A Qualcomm Company"

需要标注的一处不确定：**39 亿这个金额高通官方公告里没披露**，是媒体根据 SEC 文件（最多增发 1920 万股高通股票）推算的。

把收购放回时间线，这次开源的动机就清楚多了。Modular 融过三轮共 3.8 亿美元（2022 年 3000 万种子、2023 年 8 月 1 亿、2025 年 9 月 2.5 亿，估值 16 亿）。HN 上关于收购性质吵得很凶：

> jillesvangurp：「If I read between the lines here what happened is the VC money ran out and they arranged some kind of acquihire. That obviously raises a lot of questions about what will happen to Mojo」

> willseth 反驳：「$3.9 billion is not an acquihire!」

39 亿显然不是 acquihire。但 jillesvangurp 后半句的担心是合理的：Mojo 接下来会怎样？

从公开信息看，答案相当明确：**语言免费送，钱在上面三层收。**

1. **Modular Cloud** —— ModCon 宣布 GA，共享端点按 token 计费 + 专属部署，OpenAI 兼容 API
2. **MAX** —— 保持 Modular Community License，走 source-available + 联盟计划，企业授权
3. **对高通的战略价值** —— 一套不依赖 CUDA 的跨硬件 AI 软件栈。ModCon 同时宣布支持 AWS Trainium、Google TPU、Qualcomm Cloud AI 100

第 3 条才是 39 亿的定价依据。**编译器开源在这个框架里不是慷慨，是获客成本**——语言的采用率越高，上面那两层的生意越好做，同时高通也就越有底气说自己有条反 CUDA 的路。这对使用者其实是好消息（免费、Apache 2.0、不可撤回），只是别把它读成理想主义。

## 五、现在该不该上手

把生产就绪度的事实摆齐：

| 维度 | 现状 |
|------|------|
| 版本 | Mojo 1.0，2026-08-11 发布（随 Modular 26.5） |
| 稳定性承诺 | 「During the 1.x timeframe, changes should primarily be additive... **Breaking changes may still be made**, but will be managed with care」 |
| 平台 | 仅 macOS + Linux 原生；**Windows 只能 WSL**（原生支持已宣布与微软合作，但尚未落地） |
| 安装 | `uv pip install mojo`（以 Python wheel 分发），社区包走 `modular/modular-community` |
| 社区 | 官方称标准库开源以来近 200 位贡献者、1100+ PR、改动 20 万行；仓库 27,056 stars（实测） |
| 生产案例 | Modular 自用（MAX 和 Modular Cloud 的基础）；MiniMax 是 Modular Cloud 的旗舰客户。**Modular 之外、用 Mojo 语言本身写大规模生产代码的独立公司案例，我没查到** |

注意最后一行和倒数第二行的落差：GitHub 上 27K stars，官方能点名的语言级生产用户基本只有自己。这不是黑点，是所有新语言在这个阶段的常态——但它决定了你现在投入的性质是**下注**，不是**选型**。

一个 1.0 版本没有原生 Windows 支持，也确实招来了吐槽（HN 用户 vovavili："A bit odd to have a 1.0 release without Windows support."）。

我的建议按人群分：

**现在就值得花时间的：** 写 GPU kernel、做推理引擎、在异构硬件上做算子的人。这是 Mojo 唯一没有替代品的场景——Python 语法写 kernel、编译期元编程、一套代码跨 CPU/GPU/TPU/Trainium。有高通兜底之后，这条线的延续性反而比大多数创业公司的语言更可信。

**可以观望到年底的：** 想拿它加速普通 Python 业务代码的人。等两件事落地——编译器开始接受外部贡献（官方说今年年底），以及原生 Windows。在那之前，你的 NumPy 不会变快，重写才会。

**暂时可以完全忽略的：** 想找「无痛提速现有 Python 项目」银弹的人。那句话已经从官方路线图里删掉了，就别再按它做规划了。

最后留一个过滤器，下次再看到任何语言/框架的性能新闻都能用：

1. **基线是什么？** 是同等优化水平的对手，还是没优化过的对照组？
2. **这个数字是这次公告说的，还是三年前的旧素材被重新贴上来？**
3. **官方是不是悄悄改过定位？** 去翻 roadmap 和 FAQ 的历史版本，改口通常不发公告。

这三个问题，今天这条新闻三项全中。

---

**参考链接**

- [Modular 官方：Mojo is now open source](https://www.modular.com/blog/mojo-open-source)
- [Modular 官方：Mojo 1.0 发布公告](https://www.modular.com/blog/modular-26-5-mojo-1-0-is-here)
- [Modular 官方：ModCon 2026 公告汇总](https://www.modular.com/blog/modcon-announcements)
- [GitHub: modular/modular（编译器在 KGEN 目录）](https://github.com/modular/modular)
- [Mojo Roadmap：superset 改口原文](https://mojolang.org/docs/roadmap/)
- [Mojo 文档：Python 互操作](https://docs.modular.com/mojo/manual/python/)
- [68000x 原文 Part 3（Wayback 存档）](https://web.archive.org/web/2024/https://www.modular.com/blog/mojo-a-journey-to-68-000x-speedup-over-python-part-3)
- [HN 讨论：Mojo 1.0（428 分）](https://news.ycombinator.com/item?id=49261128)
- [HN 讨论：Mojo is now open source（125 分）](https://news.ycombinator.com/item?id=49348079)
- [高通完成收购 Modular](https://www.modular.com/blog/qualcomm-completes-acquisition-of-modular)
- 站内相关：[「Codex 提速 232 倍」的真相](/articles/codex-autoresearch-gpu-kernel-232x) · [benchmark 是怎么失去测量能力的](/articles/benchmarks-stopped-measuring)
