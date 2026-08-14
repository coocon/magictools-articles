---
title: "DeepSeek 发错了模型"
slug: deepseek-v4-pro-0813-incident
summary: "官测 Terminal-Bench 87.9 只差 Fable 5 零点一分，实测 Artificial Analysis 综合却仅 53 分，当天下午公告即被撤下。扒开 Hugging Face 提交记录与 API 指纹，这是一次配置错位的发布事故，而非模型翻车。"
category: ai-tutorials
tags: [DeepSeek, LLM, MoE, Infra, 模型发布, 事故复盘]
coverImage: ""
status: draft
locale: zh
source: authored
translationSlug: deepseek-v4-pro-0813-incident-en
---

# DeepSeek 发错了模型

> "DeepSeek 依旧延续了他们'不上发布会、不发海报'的极简风格，悄悄更新了 API 文档。"
> —— AI 前线《DeepSeek V4 Pro 正式版突遭撤回》

---

2026 年 8 月 13 日，DeepSeek-V4-Pro-0813 上线。官方跑分惊艳，实测口碑崩塌，当天下午官网撤下公告。24 小时里，梁文锋从"梁神"变成"牢梁"。

但把 Hugging Face 的提交记录、config.json 的 diff、API 返回的 system_fingerprint 三条线索并排放，你会发现故事根本不是"模型不行"。是**发错了东西**。

以下 10 条，是我按时间线和一手材料扒出来的技术复盘。文中的 HF 提交时间和 config 数值，是我在 8 月 14 日直接调 Hugging Face API 核过的。

---

## 1. 一次没有发布会的发布，也没有回滚公告

DeepSeek 的发布风格一直是改文档、改价格页，不发博客。这次也一样：API 文档里模型版本悄悄变成 `DeepSeek-V4-Pro-0813`，模型调用名不变，仍是 `deepseek-v4-pro`。

问题是，**同一套极简风格，用在出事的时候就变成了黑箱**。8 月 13 日下午官网 banner 和开放平台公告被撤下，没有任何说明。开发者只能靠猜。

**我的看法**：不开发布会是风格，出事不说话是问题。当一个模型名 `deepseek-v4-pro` 背后可以在几小时内换成不同的权重和推理栈，"版本号不变"就不是简洁，是把风险转嫁给了调用方。

---

## 2. 87.9 和 53，测的根本不是同一件事

官方数字是真的，第三方数字也是真的，两者不矛盾：

> Terminal Bench 2.1：DeepSeek-V4-Pro-0813 = 87.9，Fable-5 (w/ fallback) = 88.0
> —— DeepSeek 官方 changelog / Hugging Face 模型卡

而 Artificial Analysis 的 Intelligence Index v4.1.1 给出的综合分是 **53**，只比 V4-Flash 高 1 分。这个 Index 是 9 项评测的合成：GDPval-AA v2、τ³-Banking、Terminal-Bench v2.1、SciCode、HLE、GPQA Diamond、CritPt、AA-Omniscience、AA-LCR。同一榜单上 Claude Opus 5 是 63，Fable 5 是 62，GLM-5.2 和 V4 Pro 并列 53。

**单项 Agent 任务逼近 SOTA，综合智能落后 10 分——这两件事可以同时成立。**

还有个更隐蔽的坑：Terminal-Bench 2.0 和 2.1 不是一个东西。有媒体表格里 V4 Pro 的 Terminal Bench 是 67.9，那是 2.0 版本的数字，和官方 2.1 的 87.9 摆在一起比，纯属误导。

**我的看法**：官方选 Agent 类基准，第三方测综合智能，这是立场差异不是造假。真正该骂的是把跨版本、跨框架的分数放进同一张表——**benchmark 的版本号和 harness 配置，和分数本身一样重要**。DeepSeek 官方也在脚注里写了：用的是 DeepSeek Harness 极简模式、`max` 推理强度、`temperature=1.0, top_p=0.95`。换个 harness，分数不可复现。

---

## 3. 小弟装大哥：config 参数和 Flash 一模一样

这是整件事的核心证据。

> "最初上传的 V4-Pro-0813 配置中，hidden_size 为 4096、路由专家数为 256、隐藏层数为 43、Attention Head 数为 64；这些数值与 V4-Flash-0731 完全一致。"

社区在 Hugging Face Discussion 里提出质疑，DeepSeek 组织成员 msr2000 回复"已处理"。修正后的 config 是：`hidden_size=7168`、384 个路由专家、61 层、128 个 attention head。

我在 8 月 14 日拉了当前 config.json，确认修正值已经生效：

```json
{
  "hidden_size": 7168,
  "num_hidden_layers": 61,
  "num_attention_heads": 128,
  "n_routed_experts": 384,
  "num_experts_per_tok": 6,
  "moe_intermediate_size": 3072,
  "q_lora_rank": 1536,
  "quantization_config": { "quant_method": "fp8", "fmt": "e4m3", "weight_block_size": [128, 128] }
}
```

**我的看法**：V4 系列 Flash 和 Pro 共用一套 modeling 代码，只在宽度、深度、专家数上分叉。这种设计很优雅，但代价是——**config 写错不会崩，会静默地按错误结构去分派层**。一个 43 层的壳套 61 层的权重，加载器不一定报错，输出一定不对。这是最难查的一类 bug：没有 stack trace，只有"感觉变笨了"。

---

## 4. 不只是改 json，权重也重传了

如果只是 config 写错，改一行就完事。但事实是二进制也动了：

> "翻开关键的权重分片（model-00035 至 model-00039.safetensors）对比记录，其 SHA256 校验码全部发生改变，文件体积也出现了微妙的变化。"

我把 HF 仓库的完整提交历史拉了下来（UTC 时间，括号内为北京时间）：

| 时间 (UTC) | 提交 | 提交者 |
|---|---|---|
| 08-13 03:05 (11:05) | initial commit | msr2000 |
| 08-13 03:51 (11:51) | Add files using upload-large-folder tool ×2 | msr2000 |
| 08-13 12:25 (20:25) | **Release DeepSeek-V4-Pro-0813** | msr2000 |
| 08-13 14:18 (22:18) | **Update config.json** | msr2000 |
| 08-13 14:53 (22:53) | Add files using upload-large-folder tool ×2 | msr2000 |
| 08-13 16:28 (次日 00:28) | Update vLLM recipe link (#8) | GeeeekExplorer 等 |

正式 Release 之后不到两小时改 config，再过半小时连续两次大文件重传。**这是抢修的节奏，不是发版的节奏。**

**我的看法**：注意 initial commit 在北京时间 11:05，而 API 侧的切换更早。也就是说，**API 已经在服务流量了，开源权重还在往上传**。权重、config、线上推理这三者在那一整天里就没对齐过。

---

## 5. system_fingerprint 从明文变成哈希：把灯关了

最能说明"线上正在动手术"的，是 API 指纹：

> "此前，开发者可以通过 API 返回的明文指纹直接读出构建日期（20260812）、生产环境、FP8 量化以及 KV Cache 等工程细节。但在实测过程中，不到半小时，Pro 和 Flash 的系统指纹几乎在同一时间全部切成了无法解读的纯 32 位 Hash。"

Pro 和 Flash 同时变，意味着改的不是某个模型，是**整个推理后端的指纹生成规则**。构建日期 `20260812` 这个细节也印证了：线上跑的那版，是 12 号构建的。

**我的看法**：明文指纹被换成哈希，工程上完全合理（不该往外泄露构建细节），但时机太说明问题了——**在开发者正拿指纹当证据的时候，把可读性关掉了**。

给所有人的一个实操建议：**把 `system_fingerprint` 写进你的调用日志**。不管它是明文还是哈希，指纹变了就是模型变了。这是你唯一能在事后自证"不是我的 prompt 退化了"的东西。

---

## 6. 三层错位，才是"同一任务两个结果"的解释

> "极有可能是因为 DeepSeek 的前端接口、后端推理栈与权重配置三者出现了严重的错位，导致不同时间、不同入口的开发者拿到的体验完全不在同一个频道上。"

有开发者发现，同一项任务前后隔几小时测，表现明显不同。这不是玄学。前端路由、后端推理栈版本、权重/config 三层各自在变，你在 8 月 13 日不同时刻打进去的请求，命中的是不同的组合。

**我的看法**：这件事对评测生态的杀伤力，比模型本身大得多。**当天所有"实测"结论，包括吹的和骂的，都缺少一个必要前提：你测的到底是哪一版？** 没有指纹、没有时间戳的实测截图，在这种日子里等于零证据。

---

## 7. 1.6 万亿参数、893 GB、66 个分片——脆弱性是规模的函数

> "V4 Pro 的 FP8 量化版本权重高达 893 GB，由 66 个分片文件组成……底层配置或算子调度上的微小瑕疵，反映到高并发的前端，就会瞬间放大为显著的体验下滑。"

1.6 万亿总参数、单次激活 490 亿。官方 vLLM 示例是单节点 4×GB300，开 expert parallel、FP8 KV cache、`deep_gemm_mega_moe` 后端，外加 DSpark 投机解码（7 个投机 token）。SGLang 那边还要 `--speculative-algorithm DSPARK`。

**我的看法**：这个栈里任何一环配错，都不会抛异常，只会掉质量。投机解码的 draft 分支配错——吞吐正常，输出变差；MoE 路由专家数配错——加载成功，专家全选错；FP8 量化 scale 配错——数值稍微飘一点，长链 Agent 任务直接崩。

**万亿参数时代，模型能力的方差越来越多来自 Infra 而不是权重。** 这句话以前是安慰研究员的，现在是运维事实。

---

## 8. 屋漏偏逢连夜雨：涨价和事故撞在同一天

同一份 changelog 里还有这个：

> "为了更合理地分配资源，我们将采用峰谷定价，闲时价格为高峰时段的一半。新价格于 2026 年 8 月 16 日 16:00 UTC 生效。"
> —— DeepSeek API changelog, 2026-08-13

高峰时段是北京时间 9:00–12:00 和 14:00–18:00。V4 Pro 输出价从 6 元/百万 token 涨到高峰 27 元、闲时 13.5 元；缓存未命中输入 3 元涨到 9 元。整体算下来是原价的 1.5 到 12 倍。

**我的看法**：涨价本身站得住——V4 Flash 周调用量据称已破 4.66 万亿 token，算力是真紧张。但**"价格涨了 4 倍"和"模型好像变笨了"在同一天出现，用户的归因只有一个方向**。这不是公关问题，是排期问题：涨价和有风险的正式版切换，不该放在同一个 24 小时里。

---

## 9. 截至 8 月 14 日：官方没有为"撤回"单独说过话

我核了 DeepSeek 官方文档站的现状：

- **changelog 页面**：2026-08-13 那条 "DeepSeek-V4-Pro Update" 在，完整 benchmark 列表在（Terminal Bench 2.1 = 87.9 等 10 项），并明确写着"已在 APP、网页端和 API 全面推出"
- **发布公告页** `/news/news260813`：在线，含官方 benchmark 表和价格表图
- **Hugging Face 仓库**：最后修改停在 8 月 14 日 00:28（北京时间），是一个 vLLM recipe 链接的小改；当前 config 已是修正后的架构参数
- **关于 8 月 13 日下午撤下 banner 这件事**：**没有任何官方解释**

也就是说，模型确实是正式发布了、没有回滚；被撤下又恢复的是**公告本身**。而"为什么撤"，官方选择了沉默。

**我的看法**：这个处理方式很 DeepSeek——事做完，话不说。但对生产环境里已经切了流量的团队来说，一句"我们在 X 点到 Y 点之间修正了权重配置，期间请求可能受影响"的成本几乎为零，价值极高。**沉默省下的是公关成本，花掉的是信任额度。**

---

## 10. 开发者能从这件事里拿走什么

不是"别用 DeepSeek"。V4 Pro 的能力和价格摆在那里，Cybergym 83.3 甚至压过 Fable 5 的 83.1。真正的教训是**流程**：

1. **锁版本，别锁别名。** `deepseek-v4-pro` 这种不带日期的模型名，在生产环境里就是一颗定时炸弹。任何供应商，能锁日期版本就锁。
2. **把 `system_fingerprint` 记进日志。** 它变了，你的 A/B 结论就作废了。
3. **建自己的私有 eval。** 20 道来自你真实业务的题，跑一次五分钟。公开榜单告诉你模型排第几，私有 eval 告诉你今天能不能上线。
4. **发布日的 24 小时内别切生产流量。** 尤其是那种"模型名不变、悄悄换版本"的供应商。
5. **实测截图必须带时间戳和指纹**，否则在这种日子里没有任何证据价值。

---

DeepSeek 这次踩的坑，不是"模型不够强"，是"一个 1.6 万亿参数的系统，在部署链路上没有一道校验能拦住 hidden_size 写成 4096"。

而这个坑，所有做大模型的公司都会踩。区别只在于：有的公司有 staging 环境，config 对不上直接卡住不让发；有的公司靠社区在 Hugging Face Discussion 里帮忙 review。

**万亿参数时代最稀缺的能力，可能不是训模型，是发模型。**

---

*本文事实来源：AI 前线《DeepSeek V4 Pro 正式版突遭撤回》调查报道、DeepSeek 官方 API changelog（2026-08-13）、Hugging Face `deepseek-ai/DeepSeek-V4-Pro-0813` 仓库提交历史与 config.json（作者于 2026-08-14 核验）、Artificial Analysis Intelligence Index v4.1.1。技术报告：[arXiv:2606.19348](https://arxiv.org/abs/2606.19348)*
