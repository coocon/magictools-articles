# 「Codex 提速 232 倍」的真相：一场 GPU kernel 比赛的完整 harness 拆解

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/codex-autoresearch-gpu-kernel-232x?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/codex-autoresearch-gpu-kernel-232x?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：「Codex 提速 232 倍」的真相：一场 GPU kernel 比赛的完整 harness 拆解](https://tools.cooconsbit.com/zh/articles/codex-autoresearch-gpu-kernel-232x?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
