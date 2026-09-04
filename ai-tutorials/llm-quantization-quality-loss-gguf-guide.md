# 大模型量化到底损失多少精度？Q8 到 Q2 一张表说清，附 GGUF 怎么选

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/llm-quantization-quality-loss-gguf-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/llm-quantization-quality-loss-gguf-guide?utm_source=github&utm_medium=referral)**

先给经验法则，赶时间的看完这段就够：

- **Q8_0**：与 FP16 的差距小到测量误差级别，可视为无损，但体积只省一半——通常不值得，除非你内存宽裕到用不完。
- **Q6_K**：接近无损，困惑度（PPL）增幅约 0.3%。想要"几乎没损失"的安全档，选它。
- **Q5_K_M / Q4_K_M**：性价比甜点区。Q4_K_M 体积不到 FP16 的三分之一，PPL 增幅约 2-3%，日常对话、写代码基本无感。**社区默认推荐 Q4_K_M 不是没有道理的**。
- **Q3 档**：开始能感觉到变笨（PPL 增幅 8-10%），长推理、数学、代码上错误率上升。内存实在不够时的妥协选项，优先选带 imatrix 的 IQ3/Q3_K_M。
- **Q2 档及以下**：明显劣化（PPL 增幅 40%+），只适合"能跑起来看一眼"的验证用途。1-bit 档（IQ1_S）PPL 直接爆炸到基线的近 10 倍，不要抱幻想。

下面是数据依据和每一档的细节。

## 一、数据从哪来：llama.cpp 官方量化质量实测

网上讨论量化损失，多数是"我感觉差不多"式的体感帖。本文的数据来自 [llama.cpp 官方 perplexity 工具文档](https://github.com/ggml-org/llama.cpp/blob/master/tools/perplexity/README.md)里公布的系统性实测：**Llama-3-8B 全量化档位，RTX 4090 上用 Wikitext 语料测 PPL（困惑度）、KLD（KL 散度）和 token 概率偏移**，是目前公开资料里最完整的一份量化质量对照。

三个指标通俗地说：

- **PPL（困惑度）**：模型对文本的"意外程度"，越低越好。量化后 PPL 相比 FP16 涨得越多，损失越大。
- **KLD（KL 散度）**：量化前后模型输出概率分布的偏离程度，0 = 完全一致。比 PPL 更敏感，能抓到"PPL 差不多但行为已经变了"的情况。
- **Mean Δp**：预测正确 token 的概率平均掉了几个百分点，最接近"变笨了多少"的直觉。

## 二、核心数据表：Q8 到 IQ1 每一档损失多少

从官方完整表格中摘录关键档位（Llama-3-8B，FP16 基线 PPL ≈ 6.23，体积 14.97 GiB）：

| 量化档 | 体积 (GiB) | 相对 FP16 | ΔPPL | 正确 token 概率损失 (Mean Δp) |
|---|---|---|---|---|
| Q8_0 | 7.96 | 53% | +0.003 | -0.02% |
| Q6_K | 6.14 | 41% | +0.022 | -0.01% |
| Q5_K_M | 5.33 | 36% | +0.057 | -0.11% |
| Q4_K_M（imatrix） | 4.58 | 31% | +0.151 | -0.39% |
| Q4_K_M（无 imatrix） | 4.58 | 31% | +0.175 | -0.60% |
| IQ4_XS（imatrix） | 4.14 | 28% | +0.228 | -0.67% |
| Q4_0（老格式） | 4.34 | 29% | +0.469 | -1.59% |
| Q3_K_M（imatrix） | 3.74 | 25% | +0.503 | -1.20% |
| IQ3_M（imatrix） | 3.53 | 24% | +0.667 | -3.18% |
| Q2_K（imatrix） | 2.96 | 20% | +2.42 | -6.50% |
| IQ2_M（imatrix） | 2.74 | 18% | +2.37 | -6.46% |
| IQ1_S（imatrix） | 1.88 | 13% | +52～57（崩坏） | -32% |

...

---

**[👉 继续阅读全文：大模型量化到底损失多少精度？Q8 到 Q2 一张表说清，附 GGUF 怎么选](https://tools.cooconsbit.com/zh/articles/llm-quantization-quality-loss-gguf-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
