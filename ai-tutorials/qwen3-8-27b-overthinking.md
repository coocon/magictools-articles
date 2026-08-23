# Qwen3.8 27B：本地模型新标杆，但请先关掉默认推理档

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/qwen3-8-27b-overthinking?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/qwen3-8-27b-overthinking?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Qwen3.8 27B：本地模型新标杆，但请先关掉默认推理档](https://tools.cooconsbit.com/zh/articles/qwen3-8-27b-overthinking?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
