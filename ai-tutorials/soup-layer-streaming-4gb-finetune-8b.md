# 4GB 显存微调 8B 模型是真的：拆解 Soup 的 layer streaming 和它没写进标题的代价

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/soup-layer-streaming-4gb-finetune-8b?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/soup-layer-streaming-4gb-finetune-8b?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：4GB 显存微调 8B 模型是真的：拆解 Soup 的 layer streaming 和它没写进标题的代价](https://tools.cooconsbit.com/zh/articles/soup-layer-streaming-4gb-finetune-8b?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
