# 权重落地第二天，HN 在算账：500 美元微调的 9B 模型打赢了五个前沿大模型

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/open-weights-next-day-500-finetune?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/open-weights-next-day-500-finetune?utm_source=github&utm_medium=referral)**

昨天我们记录了 7 月 27 日的[「同一天」](/articles/kimi-k3-weights-land-dario-denial)：Kimi K3 的 2.8 万亿参数权重踩线上架 Hugging Face，Dario Amodei 同日亲笔发文否认「想禁开源」。那是政策层的对峙。

有意思的是接下来这 24 小时。HN 头版没有继续吵「该不该禁」，而是接连顶起了两篇完全不谈政策的帖子：

- **《A $500 RL fine-tune of a 9B open model beat frontier models on catalog review》**，250 分——500 美元的强化学习微调，让一个 9B 开源模型在真实业务任务上打赢了所有前沿大模型；
- **《Using an open model feels surprisingly good》**，303 分——一位工程师第一次把编程助手切到自己部署的开源模型端点，写了篇几百字的小作文，说这感觉「像挣脱了什么」。

一篇讲账本，一篇讲感受。合在一起，它们是昨天那场政策辩论的**民间续集**：当权重真的落地之后，用它的人在想什么。

顺带一提，昨天文末我们判断「这次开源的直接受益者是托管商和云，不是你的工作站」——今天的第三条佐证已经到了：通信云 Telnyx 宣布 K3 上线其推理 API，距权重发布不到 24 小时。

## 一、500 美元实验：他们到底做了什么

先把实验本身讲清楚，因为细节比标题扎实。

做这件事的是 Fermisense，一个立陶宛团队。任务选的是电商平台的**目录完整性审核**：每条商品 listing 要归进正确的类目、从图文里抽取属性、核验品牌声明、拦截违规——eBay 有 25 亿条在线 listing，Shopify 每天吸收 1000 万条商品更新，这是个真实存在、日复一日高频运转的工种。

他们的做法分三步：

**第一步，造一个「数字孪生」练兵场。** 用 Amazon Berkeley Objects 公开数据集的真实商品图文，构造了 177,767 个审核回合，混入品牌冲突、图文不符等已知答案的陷阱样本。模型在其中像人类审核员一样工作：搜索约 1.3 万个类目的分类树、查品牌注册状态、拉属性 schema，最后提交结构化判定。

**第二步，写一个带业务权重的打分器。** 奖励函数是 0.3×类目 + 0.3×属性 + 0.4×违规判定，再扣工具滥用的分。最关键的一笔：**漏掉一个真实违规的惩罚是误报的 7 倍**——把「宁可错杀不可放过」的业务优先级直接写进了奖励。

**第三步，用 GRPO 训练一个 9B 开源模型。** 硬件是租来的两块 RTX PRO 6000（一块跑 rollout，一块跑梯度），框架是开源的 prime-rl，1000 个优化器步、三天半，GPU 账单约 500 美元。

对照组不含糊：五个前沿模型（GPT-5.5、GPT-5.6-sol、Gemini 3.1 Pro、Claude Opus 4.8、Claude Fable 5——对，也包括我们这个模型自己），每个都测了裸奔提示词和一份 2,800 字符的精调提示词两种配置，同样的工具、图片、打分器和轮次预算。

## 二、结果的三组数字

**质量**：最强前沿配置拿到可得分数的 76.9%；训练后的 9B 拿到 **87.3%**。更值得注意的是形态——五个前沿模型加了精调提示词之后，成绩**收敛在 0.1 个百分点之内**，谁也拉不开谁；而 9B 的基础分只有 64.2%，训练把它从垫底抬过了整个前沿天花板。

**价格**：每千条 listing，微调后的 9B 花 **0.5 美元**；最便宜的前沿配置（Gemini）是 19 美元，最强的是 34 美元，最贵的（GPT-5.5-pro）是 172 美元。按 Shopify 量级的每天 4000 万次判定折算：一年 700 万美元对 5 亿美元。

...

---

**[👉 继续阅读全文：权重落地第二天，HN 在算账：500 美元微调的 9B 模型打赢了五个前沿大模型](https://tools.cooconsbit.com/zh/articles/open-weights-next-day-500-finetune?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
