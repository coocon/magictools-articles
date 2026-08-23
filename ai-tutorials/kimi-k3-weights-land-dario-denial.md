# Kimi K3 权重踩线上架，同一天 Anthropic CEO 亲自发文否认「想禁开源」

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/kimi-k3-weights-land-dario-denial?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/kimi-k3-weights-land-dario-denial?utm_source=github&utm_medium=referral)**

先把上一篇的悬念收掉。

两天前我在[《三个时钟》](/articles/kimi-k3-weights-three-clocks)里记录过：截至 7 月 27 日 01:08 UTC，Hugging Face 上 moonshotai 的 K3 仓库还不存在，而 Moonshot 自己的措辞是权重将在「7 月 27 日之前」（by July 27）放出——所以当时还谈不上违约，他们还剩 23 个小时。

结果是：**Moonshot 踩线交付了。**

7 月 27 日当天，`moonshotai/Kimi-K3` 仓库上线，HN 帖子冲到 **1314 分、516 条评论**，是那两天全站第一。评论区有人挂了个倒计时页面等开闸，像守跨年。

然后，同一天，另一件事发生了：**Dario Amodei 在 Anthropic 官网发表了一篇亲笔署名的立场文**，标题是《Our position on open-weights models》（我们对开放权重模型的立场）。文章第一段就直奔主题：

> 「Anthropic 从未主张禁止开放权重模型。」

一家以安全立场著称的头部实验室 CEO，在史上最大开源权重发布的当天，亲自出来发否认信——这两件事放在一起，就是这个周末 AI 行业的全部剧情。

这篇文章做两件事：先核实 K3 这次发布到底放出了什么、怎么读它的评测表；再逐条转述 Dario 实际说了什么、以及 HN 的 665 条评论为什么大面积不买账。

## 一、权重真的落地了：你拿到的是什么

从模型卡和技术报告看，这次放出的不只是一个权重文件：

- **规模**：2.8 万亿总参数、104B 激活参数的 MoE，896 个专家每 token 选 16 个（外加 2 个共享专家）。官方称之为「世界上第一个开源 3T 级模型」。
- **架构**：93 层里 69 层用 Kimi Delta Attention（KDA）、24 层用门控 MLA，配上 Attention Residuals；原生多模态（4.01 亿参数的 MoonViT-V2 视觉编码器），上下文窗口 100 万 token。
- **精度**：MXFP4 权重 / MXFP8 激活，而且是量化感知训练（QAT）的原生产物——不是发布后再压的。整包下载约 1.5TB。
- **配套 infra 同步开源**：MoonEP（专家并行）、FlashKDA（KDA 内核）、AgentEnv（agent 环境）。《三个时钟》里提过 KDA 与常规 prefix caching 不兼容、「配套推理代码没就位就发权重等于发一个跑不起来的文件」——这次他们把配套一起发了。

**许可证有一个不是 MIT 的地方**，值得原文核对：如果你用 K3 经营模型即服务（MaaS）业务，且任意连续 12 个月总收入超过 **2000 万美元**，需要与 Moonshot 另签协议；月活超 1 亿或商业产品收入超 2000 万美元的，需在产品中标注「Kimi」。对绝大多数研究者和中小团队没有影响，但托管商不能装作没看见这条。

至于「开源了人人都能跑」——不存在的。MXFP4 原生也意味着约 1.5TB 显存起步，HN 上有算力从业者估算：8 张 B200 是理论下限，考虑上下文和吞吐，现实配置是 16 张。**这份权重的直接受益者是托管商和云厂商，不是你的工作站。**

## 二、评测表怎么读：赢在哪、输在哪、脚注里藏着什么

模型卡给出了一张几十行的对比表，对家是 Claude Fable 5、GPT-5.6 Sol、Claude Opus 4.8、GPT-5.5 和 GLM-5.2。延续[《四个被误读的数字》](/articles/kimi-k3-open-weights-four-numbers)的习惯，先看脚注再看数字。

...

---

**[👉 继续阅读全文：Kimi K3 权重踩线上架，同一天 Anthropic CEO 亲自发文否认「想禁开源」](https://tools.cooconsbit.com/zh/articles/kimi-k3-weights-land-dario-denial?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
