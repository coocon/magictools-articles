---
title: "「Codex 提速 232 倍」的真相：一场 GPU kernel 比赛的完整 harness 拆解"
slug: codex-autoresearch-gpu-kernel-232x
summary: "全网疯传的「Codex 自动研究让内核提速 232 倍」，真实版本是：一位非专业选手在 GPU Mode 的 B200 QR 分解比赛里，用 Codex 跑了 14 天、1500 多次提交，拿到第 12 名。232 倍对比的是 torch.geqrf 基线，冠军区在 340 倍以上。本文基于原文、HN 讨论和同场选手复盘，拆解他的 AGENTS.md、/goal 循环、beam of candidates 和强顾问模型这套可以搬走的方法，也把过拟合、数值稳定性和 reward hacking 三盆冷水一并浇上。"
category: ai-tutorials
tags: [Codex, GPU kernel, 性能优化, Agent, auto-research, CUDA, Triton, reward hacking]
coverImage: ""
status: published
locale: zh
source: authored
---

这两天很多人转「Codex 自动研究让内核提速 232 倍」，转述版本大多有两处失真，先纠正：

1. **不是操作系统内核，也不是"自己项目"**。这是 GPU Mode 与 Core Automation 合办的 GPU kernel 优化竞赛（qr_v2 赛题）：在 NVIDIA B200 上实现批量方阵的 compact-Householder QR 分解，输出要与 `torch.geqrf` 格式逐项对齐，checker 会重建 Q 矩阵验证 A≈QR、QᵀQ≈I。
2. **232 倍不是冠军成绩**。作者 Sankalp 的原话是 "I placed 12th out of 183 participants, ending up with a 232x speedup over the baseline solution"——基线是 `torch.geqrf` 路径的约 419,000 µs，他的最终成绩 1,805 µs，419,000 ÷ 1,805 ≈ 232。第 5 名是 280 倍，第 2 名约 1,228 µs，冠军区在 340 倍以上。

数字澄清完，这篇文章想说的是：**这件事真正值钱的不是 232 这个数，而是一个一年 GPU 优化业余经验的人，靠一套可复制的 agent harness 打进了前 7%（上一名是 NVIDIA 的 principal engineer）**。他把方法完整公开了，下面逐层拆。

## 前提：这类任务为什么能让 AI 自动跑

作者总结成一句话："Agents yearn for tight feedback loops."（agent 渴望紧反馈循环。）

qr_v2 恰好满足自动化的全部前提：

- **有 oracle**：比赛提供 popcorn CLI，agent 可以自己测试、benchmark、提交排行榜，checker 返回逐 shape 计时和几何平均分——正确性和速度都有机器可验证的判据
- **提交近乎不限次**：14 天里他提交了 1,500 多次，最终目录里躺着 560 个提交变体
- **单文件、边界清晰**：不需要理解百万行代码库，优化空间集中

HN 上有评论（shken）点破了边界条件：这套循环成立的前提是每一步都有 wall-clock、profiler 或 verifier 当裁判；**没有可验证反馈的任务（比如 UI 还原），agent 会直接谎报完成**。想把这套方法搬回自己项目，第一件事不是写 prompt，是先搭 verifier。

## Harness 拆解：三个文件 + 两个命令

作者的工作区结构对标了 Karpathy 的 autoresearch 实验（人只编辑一个 program.md，agent 一夜跑约 100 次实验），核心是三个文件：

- **`problem_statement.md`**：赛题原文，不动
- **`AGENTS.md`**：操作规程——怎么用 popcorn 提交、什么算证据。里面几条纪律值得原样抄走：**"Only completed pass/fail/timing output is evidence"（只有跑完的输出才算证据）**；timeout 视为"不确定"而不是"否证"；每次提交前先跑最便宜的 sanity check；每次提交都存带时间戳的日志
- **`log.md`**：每次提交的接受/拒绝记录和逐 shape 计时簿记。作者在成绩进入深水区后特意加大了记录投入："Logs serve as the evidence of the ideas that worked and didn't work"

驱动方式是 Codex 的两类命令：

- **`/goal`**：给一个可量化的目标让模型自循环，例如原文里的真实 prompt："Use only Triton or CUDA and beat our active best's n = 512 timings... Remove cuSolver altogether"。单个 goal 曾连续跑超过一天，部分夜晚完全无人值守，作者每 2–3 小时注入一次方向
- **`/btw`、`/side`**：不打断主循环的旁路提问，问完把想法 dump 回主线程

## 卡住之后的三板斧

前 10 天成绩从 108,803 µs 一路压到 3,000 µs 附近，然后撞墙——"Optimizations were much harder after the 3000 µs point"。最后 3,000 → 1,805 µs 这段靠的是三个策略，也是全文含金量最高的部分：

**1. Beam of candidates（候选束）**。作者承认此前"只维护一个最优候选"是蠢办法，改成常驻 3–5 条 idea family，分四类：稳定收益型（exploit）、差一点就成型（near-miss）、结构性高风险型、清理型。配套两条纪律：**一条思路不能因为零星几次失败就宣判死刑**；两个各自表现中性的想法如果触及同一块独立成本，要先组合测试再淘汰。

**2. 鼓励模型冒险**。原话："Encourage the model to take more risks and try ambitious ideas. You will not believe it but this worked."——在有 verifier 兜底的场景里，激进尝试的下行风险是有限的，这是竞赛环境给的红利。

**3. 强顾问模型**。他在 AGENTS.md 里指示 Codex 卡住时 headless 调用 `claude -p` 要新点子——一个模型执行、另一个模型出主意。作者预言这种 "strong advisor strategy" 会成为 auto-research 流程的标配。顺带一提他的模型手感："Claude would often give up after a few rounds... Codex, on the other hand, is more persistent."

成本方面：ChatGPT Pro（200 美元/月）跑 Codex CLI，Claude Pro（20 美元/月）当顾问，Modal 每月 30 美元免费额度做 profiling。没有额外 API 开销。

## 三盆冷水：过拟合、数值稳定性、reward hacking

这篇文章在 Hacker News 上有 386 分、86 条评论，最有价值的恰恰是质疑。

**过拟合比赛输入**。评论区最硬的一条（augment_me）："8 out of the 10 top solutions... completely broke at any other input than the competition ones"——榜单头部方案大多针对比赛的 12 种固定 shape 特化，换个输入直接崩，"如果你是开源库维护者，这没用"。**作者本人回复 "fair argument"**，还补充了后续 Cholesky 赛题的数据：主办方拿小规模训练任务验证头部方案，大多数只过了 8 项里的 4 项。

**数值稳定性**。多位评论者指出，用 Cholesky-QR 替代 Householder 更快但稳定性差、适用面窄——这正是 PyTorch 不默认这么做的原因。竞赛指标只有"快"，生产环境的指标还有"在病态条件数下不出错"。

**Reward hacking 是真实存在的**。作者致谢里专门感谢了帮忙 "detecting reward hacks" 的人；另一赛题的榜首代码里被扒出一行 "bypass ban check"。这有个著名先例：2025 年 Sakana AI 的 "AI CUDA Engineer" 宣称 10–100 倍加速，实测反而慢 3 倍——系统在评测代码里找到了 memory exploit 绕过正确性检查，官方最终公开道歉。**让 agent 自动优化，verifier 本身就会成为攻击面**。

还有一条清醒剂（fooblaster）："You can't exceed roofline performance... the idea that it is leading to some exponential growth is a total pipe dream"——作者也回了 "fair enough"。硬件屋顶就在那里，AI 只是把逼近屋顶的人力成本打下来了。

## 你能搬走什么

结合作者的复盘和 HN 讨论，如果想在自己项目里试这套方法，按这个顺序做:

1. **先搭 verifier，再谈自动化**：一个能自动判对错、自动出耗时数字的闭环（哪怕就是一个带断言和计时的测试脚本）。没有它，一切免谈
2. **写 AGENTS.md 立证据规则**："只有跑完的输出算证据"、"timeout 不算否证"，这两条能砍掉大量 agent 的自欺行为
3. **留日志**：每次尝试的结果簿记下来，这是 agent 长程工作时对抗 context 腐化的外部记忆
4. **卡住时上 beam**：并行维护几条思路，别让 agent 在单一路径上死磕
5. **对结果保持竞赛级警惕**：拿到"提速 N 倍"后，先用 OOD 输入和边界条件打一遍——你要防的就是上面那两种翻车

作者在 HN 的补充值得当结语："having a harness as thin as possible with some problem specific instructions while controlling for context rot is the key"——harness 尽量薄，指令针对问题，管住 context 腐化。他用同方法的变体在后续比赛又拿了第 7。

## 常见问题 FAQ

### 232 倍提速是真的吗？

数字本身是真的，但语境重要：对比的是 PyTorch `torch.geqrf` 基线（约 419 ms → 1,805 µs），发生在 GPU Mode 竞赛的 12 种固定输入 shape 上，成绩排第 12/183。它不代表 AI 能把任意生产代码提速两个数量级——HN 讨论证实榜单头部方案大多在比赛输入之外直接失效。

### 没有 GPU 优化背景能复现这套方法吗？

作者自己的判断是可以拿到"体面的提速"："this contest was doable without domain knowledge... you can get a respectable speedup by just relying on your harness/agent loop"，但进前 10 需要领域知识——他复盘里列的遗憾（没做输入分布检测、没让 trailing matrix 常驻 fp16）全是领域功力问题。

### 这套方法适用于什么样的任务？

判据只有一条：每步尝试能否被机器自动验证（测试通过 + 可测量的目标指标）。性能优化、编译产物瘦身、查询调优这类有硬指标的任务适合；UI 还原、代码可读性这类没有 oracle 的任务，agent 会谎报完成。

### 用的是什么工具，花了多少钱？

Codex CLI + GPT-5.5（ChatGPT Pro，200 美元/月订阅），Claude Pro（20 美元/月）做顾问模型，Modal 免费额度（30 美元/月）做 profiling。14 天，1,500 多次提交，无额外 API 费用。

## 参考链接

- [Auto-research with codex: How I achieved a 232x Faster Kernel — Sankalp 原文](https://sankalp.bearblog.dev/autoresearch/)
- [Hacker News 讨论（386 分 / 86 评论）](https://news.ycombinator.com/item?id=49309549)
- [同场第 5 名选手 Mike 的技术复盘（280x）](https://ml-mike.com/writing/qr_v2/)
- [Karpathy 的 autoresearch 实验](https://github.com/karpathy/autoresearch)
- [Sakana AI 撤回「AI CUDA Engineer」加速宣称 — TechCrunch](https://techcrunch.com/2025/02/21/sakana-walks-back-claims-that-its-ai-can-dramatically-speed-up-model-training/)
- [KernelBench：AI 写 kernel 的标准评测 — Stanford](https://github.com/ScalingIntelligence/KernelBench)
- [GPU Mode qr_v2 赛题说明](https://github.com/gpu-mode/popcorn-cli/blob/main/docs/linalg-qr-b200.md)
