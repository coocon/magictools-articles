# Mojo 开源了编译器，也悄悄改掉了「Python 超集」那句话

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/mojo-open-source-68000x-truth?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/mojo-open-source-68000x-truth?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Mojo 开源了编译器，也悄悄改掉了「Python 超集」那句话](https://tools.cooconsbit.com/zh/articles/mojo-open-source-68000x-truth?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
