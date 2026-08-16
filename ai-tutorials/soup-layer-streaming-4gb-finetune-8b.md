---
title: "4GB 显存微调 8B 模型是真的：拆解 Soup 的 layer streaming 和它没写进标题的代价"
slug: soup-layer-streaming-4gb-finetune-8b
summary: "GitHub 项目 Soup 宣称一个 YAML 文件就能在 4GB 显存的笔记本 GPU 上微调 Llama-3.1-8B。我们读了它的核心代码、benchmark 文件和 HN 讨论：宣称成立——RTX 3050 Laptop 实测峰值 3.32GB、119.6 tok/s，比业界 QLoRA 的 6.6–8GB 下限低了一半。本文讲清 layer streaming 的原理（冻结底座放内存、双缓冲预取、每层每步读两次）、上手配置，以及边界条件：1.43 倍速度代价、16GB 内存下限、架构白名单，和两起已修复的静默错误。"
category: ai-tutorials
tags: [LLM 微调, LoRA, QLoRA, 显存优化, layer streaming, Soup, 开源项目, bitsandbytes]
coverImage: ""
status: published
locale: zh
source: authored
---

「4GB 显存微调 8B 模型」这种标题，默认应该按营销话术处理——业界共识是 QLoRA 微调 8B 至少要 6.6–8GB 显存（Unsloth 官方给的最优情况是 8GB，第三方在 4090 上实测 seq 2048 峰值 6.6GB）。所以我们把 [Soup](https://github.com/MakazhanAlpamys/Soup) 这个仓库从 README、benchmark 文件一路读到核心代码，结论是：**宣称成立，但边界条件很多，而且作者把代价写得比大多数项目诚实**。

先给硬数字。作者在自己的开发机（RTX 3050 Laptop，4GB 显存，Windows 11）上的实测：Llama-3.1-8B-Instruct，NF4 量化，LoRA，batch 1，seq 512——**峰值显存 3.32GB，吞吐 119.6 tok/s，GPU 利用率 100%**。同配置在 H100 上复现出 113 tok/s 和相同的 3.32GB（H100 反而略慢，说明瓶颈根本不在 GPU 算力）。还有一个免费 Colab T4 就能跑的复现 notebook，用 `set_per_process_memory_fraction` 把进程硬限在 4.00GB，8B 照样跑通。

## Layer streaming 的原理：显存里只留一层

常规微调（包括 QLoRA）要求整个模型权重常驻显存，这就是 8B 模型 6.6GB 起步的原因。Soup 的 `stream_layers: true` 换了个思路：

- **冻结的底座权重放 CPU 内存**（尽量做 page-lock/pinned，内存不够时降级到 NVMe 磁盘；SATA/机械盘会被直接拒绝），显存里常驻的只有 LoRA adapter、它的梯度和优化器状态
- 预分配 2–8 个显存缓冲区（默认双缓冲），在专用 CUDA stream 上做异步预取：forward 算第 i 层时预取第 i+1 层，backward 算第 i 层时预取第 i-1 层——**权重搬运和计算重叠**，这是 GPU 利用率能到 100% 的关键
- NF4 路径下，底座离线量化一次并分片缓存，缓存 key 带量化方式和源 checkpoint 指纹，防止流错字节

天下没有免费显存，代价藏在两个物理事实里：

1. **每层每步要从内存读两次**（forward 一次，backward 重算一次——streaming 强制开 gradient checkpointing）。代码注释原话很直白："dL/dx = Wᵀ·dL/dy, this is physics, not an implementation detail"
2. **embeddings 和 lm_head 常驻且不量化**——8B 峰值 3.32GB 里这部分占 2.10GB。这也解释了为什么 8B 贴着 4GB 天花板、作者说 14B 他没试：能省的只有 transformer 层，词表相关的部分省不掉

有个细节能看出工程完成度：训练前有 VRAM pre-flight 预测器，预测超预算**直接拒绝运行**而不是让你 OOM（Windows 下超显存不报错而是静默降速 9 倍，这个预测器就是为了防它）；预测模型拟合自 10 次真实运行，最差误差 0.85%，只朝安全方向偏。

## 上手：确实只要一个 YAML

安装是 `pip install soup-cli`（当前 v0.73.2）。README 里的最小流式配置：

```yaml
training:
  stream_layers: true      # 底座流式进出显存，只有 adapter 在训练
  quantization: 4bit       # NF4，底座缩小约 4 倍，8B 才塞得进 4GB
  batch_size: 4            # 大 batch 摊薄每层的权重读取成本
  stream_source: auto      # 内存装得下用内存，装不下用 NVMe
  seed: 1234
```

配上 `base`（模型名）、`data`（jsonl + alpaca 格式）、`lora`（r/alpha）几段就能跑。两个实用结论直接抄作者的 benchmark：**同等有效 batch 下，直接加大 batch_size 比梯度累积快 2.52 倍**（1378 vs 540 tok/s，代价是显存从 0.85GB 涨到 2.28GB）；1M token 的训练量在 8B 上约 2.3 小时（作者标明是推算值）。

## 边界条件清单（决定你能不能用）

这部分比原理重要，逐条列：

- **速度代价约 1.43 倍**（vs 常驻训练，0.5B 上实测——唯一能公平对比的尺寸），另有逐层 NF4 反量化约 9.8% 的开销。预印本 v3 有个值得尊敬的更正：作者原以为瓶颈是 host-to-device 传输，实测证伪（删掉全部拷贝只快 1.4%），真正的流式专属开销是反量化
- **系统内存是新瓶颈**：8B 的 NF4 底座要 page-lock 约 3.6GB 内存，作者明说 **16GB 系统内存是下限**
- **架构白名单**：llama / qwen2 / qwen3 / mistral / gemma 系 / phi 系；任务仅 sft / dpo / orpo / simpo / kto；**grpo 和 ppo 永久不支持**——RL 的 rollout 生成阶段每个 token 都要重读全部层，直接摧毁流式的摊销前提
- **不兼容 Unsloth/mlx 后端**，也不兼容 DoRA、VeRA、packing。也就是说你放弃了 Unsloth 的速度优化来换显存
- **功能还是 BETA**，且实质是单人项目（745/756 commits 来自作者一人）
- **头条数字有一处时效瑕疵（README 自己标注的）**：119.6 tok/s 测于 v0.72.2；v0.73.0 修了一个正确性 bug 后（32B 上吞吐 -4.8%）没在 4GB 卡上重测

## 为什么这个项目值得单独写一篇：翻车记录全部公开

读它的 benchmark 目录像读事故报告汇编，这在开源项目里非常罕见：

- **v0.72.0 的静默错误**：流式训练出的 adapter 键名多了一段 `.inner.`，所有加载器会**静默加载出未微调的原模型**——PEFT 只发一个 UserWarning。v0.72.1 修复
- **Issue #331 的梯度错误**：NF4 且单层超过约 165MiB（32B/72B 级别）时，forward 逐位正确、loss 曲线正常，但**梯度悄悄全错**。根因是 bitsandbytes 的 `MatMul4Bit` 把权重塞在 ctx 普通属性上绕过了 `save_for_backward`，和 Soup 的池化缓冲区产生 aliasing。v0.73.0 修复
- 与常驻训练**逐位一致（bit-exact）**的验证覆盖 9 个架构族 × 2 种精度，max abs logit diff = 0.0——上面两个 bug 就是被这类验证抓出来的

Show HN 帖（138 分）里还有两段插曲：simonw 质疑示例目录里训练数据只有几行，作者回应那是格式样例，并当场发现 8 个示例配置里 7 个用了旧 schema 根本解析不了，随即修复加测试；作者早期用 LLM 代写英文回复被社区抓包（近半评论被标 dead），承认后改为亲笔——他的母语是哈萨克语和俄语。

最有说服力的是作者自己的劝退："**不要为此去买 4GB 卡**。如果买卡，买大显存的；模型能常驻显存就别开 streaming。"这个功能的真实定位是：**你手上恰好只有 4–6GB 的卡，又想在本地动一动 8B 模型**。

## 常见问题 FAQ

### 4GB 显存微调 8B 到底是不是噱头？

不是。RTX 3050 Laptop 4GB 实测峰值 3.32GB、119.6 tok/s，H100 复现和 Colab 可验证 notebook 三方一致，且与业界 QLoRA 下限（6.6–8GB）对照后确实是突破。但条件固定：NF4 量化 + LoRA + seq 512 + 白名单架构 + 16GB 系统内存，速度比常驻训练慢约 1.43 倍。

### 和 Unsloth / QLoRA 是什么关系？

QLoRA 解决"权重太大"（4bit 量化），但整个模型仍需常驻显存；Unsloth 在此基础上做速度优化，8B 最低约 6.6–8GB。Soup 的 layer streaming 解决"常驻"本身——显存里只留一层 + adapter，所以能把下限压到 3.32GB，代价是与 Unsloth 加速不兼容且更慢。显存够 8GB 的话，Unsloth 路线仍然是更快的选择。

### 训练质量会打折吗？

机制上不会：layer streaming 与常驻训练做过逐位一致验证（bit-exact，logit 差为 0）。风险在工程实现——它出过两次"训练看起来正常、结果悄悄错了"的 bug（均已修复并公开归档），用的时候建议训完做一次小样本推理对比，确认 adapter 真的生效。

### 什么情况下不该用它？

要跑 GRPO/PPO（永久不支持）；模型能常驻显存（作者自己建议别开 streaming）；用非白名单架构；系统内存低于 16GB；以及对训练速度敏感的生产管线。

## 参考链接

- [Soup 仓库 — GitHub](https://github.com/MakazhanAlpamys/Soup)
- [Show HN 讨论（138 分）](https://news.ycombinator.com/item?id=49166984)
- [soup-cli — PyPI](https://pypi.org/project/soup-cli/)
- [预印本：Exact Layer Streaming: LoRA Fine-Tuning of an 8B Model on a 4 GB Laptop GPU — Zenodo](https://doi.org/10.5281/zenodo.21918325)
- [Unsloth：Llama 3.1 8B 微调显存基线](https://unsloth.ai/blog/llama3-1)
- [Soup 官网](https://trysoup.dev/)
