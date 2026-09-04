# 24GB Mac mini 能跑多大的本地大模型？内存账、实测速度与提速方案汇总

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/mac-mini-24gb-local-llm-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/mac-mini-24gb-local-llm-guide?utm_source=github&utm_medium=referral)**

直接回答：**24GB 统一内存的上限是 27B 级模型的 4-bit 量化**——不是理论推算，我们在一台 M4 丐版 Mac mini 上把 Qwen3.8-27B（4-bit，模型本体约 15GB）连同 2B 草稿模型完整跑通，峰值内存 19.4GB，前提是关掉浏览器等大内存应用。速度上，纯自回归 6.5 tok/s"勉强能看"，开投机解码后 11.7-12.2 tok/s 接近正常阅读速度。

这篇是本站 Mac mini 本地部署系列的汇总页：内存账怎么算、每个参数量级什么体验、三条提速路线哪条真有用，每个结论都链接到对应的完整实测。

## 一、内存账：24GB 实际能给模型多少

统一内存是 CPU/GPU 共享的，系统本身和常驻应用要吃掉一块。我们的实测经验：**给模型和推理栈留出 19-20GB 是可行的，但需要主动关闭大应用**；常驻 Chrome + IDE 的日常状态下，安全预算在 16GB 左右。

模型文件体积的估算公式（含格式开销，[依据见量化专文](/zh/articles/llm-quantization-quality-loss-gguf-guide)）：

> 体积 ≈ 参数量 × 每参数字节数（Q4_K_M 约 0.6 字节，Q6_K 约 0.8，Q8_0 约 1.06）

在此之上还要加 KV cache（随上下文长度线性增长）和运行时开销。按这笔账，各量级在 24GB 上的处境：

| 模型量级 | 推荐量化 | 模型体积 | 24GB 上的体验 |
|---|---|---|---|
| 7-9B | Q8_0 / Q6_K | 6-10GB | 舒适区：高量化档 + 长上下文 + 不用关应用 |
| 13-14B | Q6_K / Q5_K_M | 8-11GB | 从容：还有余量给长上下文 |
| 20B 级 | Q5 / Q4 | 11-14GB | 可行：适度控制上下文长度 |
| 27-32B | Q4_K_M / 4-bit | 15-18GB | 天花板：能跑，需关大应用、控上下文（27B 实测峰值 19.4GB） |
| 70B+ | — | Q4 也要 40GB+ | 跑不动，别试了 |

一个反直觉的提醒：**内存不够时，降量化不如换小模型**。27B 压到 Q2 的概率分布已经严重变形，通常不如 14B 的 Q5——这背后的数据见[量化精度损失专文](/zh/articles/llm-quantization-quality-loss-gguf-guide)。

...

---

**[👉 继续阅读全文：24GB Mac mini 能跑多大的本地大模型？内存账、实测速度与提速方案汇总](https://tools.cooconsbit.com/zh/articles/mac-mini-24gb-local-llm-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
