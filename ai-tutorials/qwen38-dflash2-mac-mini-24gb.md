# 24G Mac mini 跑 Qwen3.8-27B + DFlash 2：实测 1.8x，以及为什么到不了官方的 3x

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/qwen38-dflash2-mac-mini-24gb?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/qwen38-dflash2-mac-mini-24gb?utm_source=github&utm_medium=referral)**

2026 年 8 月是本地模型玩家的好月份：8 月 14 日 Qwen3.8-27B 开源权重，8 月 18 日 [Inco AI 发布 DFlash 2](https://inco.ai/blog/dflash2/)，并直接放出了对应 Qwen3.8-27B 的草稿模型。官方博客里的数字很漂亮：SGLang 上 batch size 1 达到自回归解码 **2.7–3.4 倍**的吞吐。

问题是，那是数据中心 GPU 上的数字。我手上是一台 24GB 内存的 Mac mini。这篇文章回答三个问题：**能不能跑起来？实际快多少？差距差在哪？**

先给结论：能跑，稳定 **1.8–1.9 倍**（6.5 → 11.7–12.2 tok/s），全程三轮对照实测，波动 5% 以内。27B 模型从"勉强能看"提到"流畅阅读"，这台机器的形态被这两个 8 月的发布实质性改变了。

## 测试机器（如实交代）

| 项目 | 配置 |
|---|---|
| 机型 | Mac mini（Mac16,10），2024 款丐版档位 |
| 芯片 | Apple M4（基础版，非 Pro/Max） |
| CPU | 10 核（4 性能 + 6 能效） |
| GPU | 10 核，Metal 4 |
| 内存 | 24GB 统一内存（M4 基础版带宽规格 120GB/s） |
| 系统 | macOS 26.3.1 |

强调"基础版 M4"是因为它直接决定结果：本地 LLM 解码是内存带宽瓶颈型任务，M4 Pro（273GB/s）和 M4 Max 的绝对速度会明显更高，但**加速比**（本文的主角）主要由算法和量化配置决定，参考价值是通用的。

## 为什么走 MLX，而不是 vLLM / SGLang

DFlash 2 官方支持四条路：SGLang、vLLM、llama.cpp（PR 阶段）和 **MLX**。前两条都是 CUDA 生态，Mac 上不用想。而 [z-lab/dflash](https://github.com/z-lab/dflash) 官方仓库直接内置了 Apple Silicon 的 MLX 后端，README 明确列出 DFlash 2 对 Qwen3.8-27B 的 MLX 支持——这不是社区 hack，是第一方路径。

## 部署：四条命令

```bash
# 1. 环境（Python 3.12：3.14 对 MLX 生态还太新，别冒险）
mkdir -p ~/llm/dflash && cd ~/llm/dflash
uv venv --python 3.12 .venv
uv pip install --python .venv/bin/python "dflash[local]"

# 2. 模型（共约 19GB 下载）
.venv/bin/hf download mlx-community/Qwen3.8-27B-4bit    # 目标模型 15G
.venv/bin/hf download z-lab/Qwen3.8-27B-DFlash2         # 草稿模型 3.85G

# 3. 验证 MLX 跑在 GPU 上
.venv/bin/python -c "import mlx.core as mx; print(mx.default_device())"
# 期望输出: Device(gpu, 0)

# 4. 生成
.venv/bin/dflash generate mlx \
    --model mlx-community/Qwen3.8-27B-4bit \
    --draft z-lab/Qwen3.8-27B-DFlash2 \
    --draft-bits 4 --block-size 5 --reasoning low \
    "你的问题"
```

...

---

**[👉 继续阅读全文：24G Mac mini 跑 Qwen3.8-27B + DFlash 2：实测 1.8x，以及为什么到不了官方的 3x](https://tools.cooconsbit.com/zh/articles/qwen38-dflash2-mac-mini-24gb?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
