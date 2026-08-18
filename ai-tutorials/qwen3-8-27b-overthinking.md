---
title: "Qwen3.8 27B：本地模型新标杆，但请先关掉默认推理档"
slug: qwen3-8-27b-overthinking
summary: "一个 17GB 的量化文件拿下 Artificial Analysis 52 分、画出本地模型史上最好的鹈鹕，但 Simon Willison 实测同一个任务默认档要 21 分钟、关掉推理只要 137 秒。这篇拆解 Qwen3.8 27B 的真实水平、「过度思考」的量化证据，以及 reasoning_effort 到底该怎么调。"
category: ai-tutorials
tags: [Qwen, 本地模型, LLM, 开源模型, Simon Willison, 模型评测]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: qwen3-8-27b-overthinking-en
---

# Qwen3.8 27B：本地模型新标杆，但请先关掉默认推理档

同一个模型，同一周里收获了两种截然相反的头条。

一边是 Artificial Analysis 的独立评测：**Intelligence Index 52 分**——上一代 Qwen3.6 27B 只有 38 分，一代跳了 14 分，一个 27B 的 dense 模型追到了旗舰 MoE 的门口（Qwen3.8-Max 也才 58 分）。

另一边是 Simon Willison 8 月 16 日的实测博客，标题毫不客气：《Qwen 3.8 27B is excellent, but it defaults to **wildly** overthinking things》——很强，但默认设置「疯狂过度思考」。

这两件事都是真的，而且互为注脚。Hacker News 上三波讨论合计超过 2400 分（发布帖 1423 分、Simon 文 751 分、AA 评分帖 305 分），这大概是今年本地模型圈最热闹的一周。

---

## 先把模型本身说清楚

Qwen3.8 27B 由阿里通义实验室于 8 月 14 日发布：27.78B 参数、**dense 架构**（不是 MoE）、多模态输入（文本/图像/视频）、原生 262,144 token 上下文（YaRN 可扩到 1M）、Apache 2.0 协议。对本地玩家最关键的一个数字：Q4_K_M 量化后**只有 17GB**，一台 24GB 显存的机器或大内存 Mac 就能跑。

官方自报的基准很凶：SWE-Bench Pro 61.7%、LiveCodeBench 90.3%、GPQA Diamond 89.2%。照例提醒：这些是厂商自报数字，部分跑在内部修改版基准上，目前没有独立复现——真正有第三方背书的是上面那个 AA 52 分。

## 21 分钟 vs 137 秒：过度思考长什么样

Simon 的测试环境是 128GB 的 M5 Max MacBook Pro 和一台 NVIDIA DGX Spark。问题出在模型的默认推理档位：`reasoning_effort` 出厂默认 **xhigh**。

后果是什么？他的招牌测试「画一只骑自行车的鹈鹕 SVG」，默认档跑了 **21 分钟**，烧掉 22,276 个 reasoning token；关掉推理之后，同样的任务 **137 秒**出结果。更荒诞的一个例子：让它「画一个圆的 SVG」，它自己脑补出了一套包豪斯风格的动画圆环设计研究——Simon 的原话是 "absolutely beautiful... which was entirely not what I had asked for!"（美极了……但完全不是我要的东西）。

他对默认设置的评价："This is a *hilarious* default. It's absolutely not a good way to run the model, especially on consumer hardware."（这个默认值很搞笑，绝对不是运行这个模型的正确方式，尤其在消费级硬件上。）

有意思的是，Artificial Analysis 的数据独立佐证了「话痨」这个判断：跑完整个 Intelligence Index，Qwen3.8 27B 生成了 **1.6 亿个 token**，而全部上榜模型的中位数是 4300 万——**接近 4 倍于中位数的输出量**。过度思考不是 Simon 的体感，是有账单的。

## 但推理也不是废物

在把默认档骂完之后，Simon 补了一个反例：他让模型给图片做 bounding box 标注并生成标注工具，关闭推理时结果「几乎能用，但框的位置是错的」；开启推理后就对了。他的原话："this is a good example of how reasoning can make a difference"。

所以正确的结论不是「推理无用」，而是**推理深度应该按任务分配，而这个模型把最贵的档位设成了出厂默认**。

## 实操：三档怎么调，速度怎么救

**推理档位**。`reasoning_effort` 有三档：`xhigh`（默认）/ `medium` / `low`，也可以完全关闭 reasoning。Simon 的明确建议："My strong recommendation: ignore that default. Run Qwen 3.8 27B on low or even no reasoning levels at first."（强烈建议无视默认值，先用 low 甚至无推理档跑。）日常问答、代码补全、格式转换用 low/off；确实需要多步推理的任务（复杂调试、空间理解类）再升 medium/xhigh。

**速度**。dense 架构吃内存带宽，这是它在本地跑不快的结构性原因——LM Studio 下 Simon 实测只有 15-30 token/s，他坦言「感觉慢……要把我从托管 API 模型那里赢回来还很难」。但有个额外的翻盘点：模型内建了 **Multi-Token Prediction（MTP）**，llama.cpp 用 `--spec-type draft-mtp` 开启后，Simon 在 DGX Spark 上实测提速约 **72%**（方法来自 Georgi Gerganov 的推文）。用 llama.cpp/llama-server 的用户别漏掉这个开关。

## 为什么这个模型重要

Simon 给这个模型的总评有两句值得抄下来。

第一句关于现在："The fact that a 17GB file can do all of this stuff on my home machines is a *miracle*."（一个 17GB 的文件能在我家里的机器上干出这一切，是个奇迹。）他还确认这是「迄今为止本地模型画出的最好的鹈鹕」，视觉 bounding box 的精度「好得惊人」，还能驱动 Pi coding agent 读 Datasette 源码并写出可用工具。

第二句关于趋势："The most important thing about Qwen 3.8 27B is **what it demonstrates**."（这个模型最重要的，是它证明了什么。）一年前这个水平的能力还需要几百 GB 的旗舰模型；现在它被压进了 17GB、Apache 2.0、任何人都能下载的文件里。dense 追平 MoE、本地追赶云端的速度，比任何一家厂商的路线图都快。

只是记得：下载完之后，第一件事是把 `reasoning_effort` 调低。好模型配错默认值，白白浪费你 21 分钟。

---

*资料来源：*
*Simon Willison：[Qwen 3.8 27B is excellent, but it defaults to wildly overthinking things](https://simonwillison.net/2026/Aug/16/qwen-38-27b/)*
*Artificial Analysis：[Qwen3.8 27B 模型页](https://artificialanalysis.ai/models/qwen3-8-27b)*
*Qwen 官方：[发布博客](https://qwen.ai/blog?id=qwen3.8) / [Hugging Face 模型卡](https://huggingface.co/Qwen/Qwen3.8-27B-FP8)*
*Hacker News：[发布帖（1423 分）](https://news.ycombinator.com/item?id=49299605) / [Simon 评测帖（751 分）](https://news.ycombinator.com/item?id=49324985) / [AA 52 分帖（305 分）](https://news.ycombinator.com/item?id=49334544)*
