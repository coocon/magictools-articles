---
title: "Claude 从零设计蛋白质：成功率翻倍"
slug: claude-protein-binder-design
summary: "Anthropic 让 Claude 独立跑完一次真实的蛋白质结合剂设计战役：15 个靶点成功 14 个，命中率 22%–35%，是行业典型水平的两倍以上，并由两家独立湿实验室验证。更关键的是，那份约 3 万 token 的提示词和全部实验数据被完整开源。"
category: ai-tutorials
tags: [Claude, Anthropic, 蛋白质设计, AI for Science, 开源, Opus, 药物研发]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: claude-protein-binder-design-en
---

# Claude 从零设计蛋白质：成功率翻倍

> 本文基于 Anthropic 2026 年研究博客《How Claude is accelerating protein design and analytical chemistry》整理，提炼 10 个观点并做解读。引语均为原文英文原句。
> 原文链接：https://www.anthropic.com/research/Claude-accelerates-protein-design

---

过去几个月，AI 在数学上刷成绩已经不新鲜了——因为数学的验证很便宜，跑个证明检查器就知道对不对。

真正难的是那些「验证很贵」的领域：生命科学。你想知道一个蛋白质设计对不对，得合成出来、送进湿实验室、等几周。

这次 Anthropic 干的就是这件事。他们让 Claude 独立跑完一场真实的蛋白质结合剂（minibinder）设计战役，把设计稿寄给两家外部实验室做物理验证，然后把结果、提示词、全部数据一次性公开。

以下是 10 个值得注意的点。

---

## 1. 22%–35% 对 10%–15%：这不是提升，是翻倍

> "Claude (Mythos Preview and Opus 4.8) designed protein binders against 15 targets, and succeeded against 14 of them. Between 22% and 35% of its individual designs bound successfully, depending on the setup, compared to the 10-15% that is typical in protein design campaigns today."

命中率（hit rate）是这行的硬通货：你设计 100 个候选，最后真能结合上靶点的有几个。

行业今天的典型水平是 10%–15%。Claude 拿到的是：多靶点 48 小时会话里，Mythos Preview 26.7%、Opus 4.8 22.6%；改成单靶点模式（每个靶点独立开 24 小时会话），Mythos Preview 冲到 35.1%。

15 个靶点里成功了 14 个，总共产出 354 个确认结合剂，来自 1320 份设计。

这个数字的分量在于：它不是把 10% 推到 12%，而是直接翻倍。在一个「候选合成成本按个计价、湿实验室排期按周计价」的行业里，命中率翻倍意味着同样的钱能验证的有效假设翻倍。

**My take：** 大多数 AI-for-Science 的成绩单要么是基准测试分数，要么是「协助专家完成了 X」。这次给的是产业界每天都在看的那个指标，而且是在真实实验里量出来的。指标选对了，说服力就完全不一样。

---

## 2. 端到端自主：人类只负责点「同意」

> "After giving Claude the prompt, we left the model to execute autonomously. We provided no additional scientific, technical, or operational guidance after we initiated the campaigns."

这句话是整篇文章最狠的一句。

人类的全部参与是：批准网络访问请求、盯着基础设施别崩、把生成的设计寄去实验室下单。科学上、技术上、操作上的指导，零。

Claude 自己做完了这些事：选择在靶点蛋白的哪个位置下手；编排多个结构设计、序列设计和共折叠模型生成候选；跑多轮计算机模拟优化；筛选出既新颖、又能表达、又能保持可溶、又能结合的候选。

> "Claude conducted all of the work that goes into designing a binder, which can take a human operator weeks."

资源配置也公开了：多靶点模式 48 小时 wall time + 最多 12500 个 H100 小时；单靶点模式每个靶点 24 小时 + 最多 2500 个 H100 小时。token 和子智能体预算不设限，fast mode 开着。

**My take：** 注意「orchestrating」这个词——Claude 用的是这个领域已经在用的公开专业模型，它没有发明新算法。它做的是编排：决定用哪个模型、怎么串、什么时候该重来一轮。这恰好是过去需要一个计算生物学专家花几周手工做的事。AI 在科研里的第一个真实位置，可能不是「发现新原理」，而是「替掉那个编排流水线的人」。

---

## 3. 验证方是外人，不是自己

> "Our external evaluators, Adaptyv Bio and Twist Bioscience, independently produced and tested Claude's designs in the lab, finding that of the 15 targets we designed against, Claude successfully designed binders against 14 of them."

Adaptyv Bio 和 Twist Bioscience 独立生产并测试了这些设计。Anthropic 不碰湿实验室那一段。

结果：至少 6 个靶点拿到高亲和力结合剂（定义是 KD < 10 nM，个位数纳摩尔），至少 4 个靶点达到或超过已发表的最佳亲和力。

亲和力为什么要紧？因为它决定药能用多小的剂量起效——剂量越小，副作用风险越低，生产成本越低。

**My take：** AI 公司自己发的 benchmark 我一律打折扣。这次不一样的地方在于，判分的是两家做蛋白质合成和测序的商业公司，用的是分子在试管里的真实行为。这类第三方物理验证在 AI 圈还很稀缺，应该成为标配。

---

## 4. RBX1：40% 对 3.7%

> "Against RBX1 (a small protein that drives the targeted destruction of specific regulatory proteins), Mythos Preview in single-target mode achieved a 40% hit rate, compared to a 3.7% hit rate among participants."

Adaptyv Bio 办过公开的蛋白质设计竞赛，参赛的是全球的人类团队和他们的工具链。

RBX1 这个靶点上，全体参赛者的命中率是 3.7%。Claude 单靶点模式下是 40%。差了十倍。

而且 Claude 排名第一的那个设计，亲和力超过了那场比赛的冠军作品——冠军是从 245 份参赛设计里选出来的。

**My take：** 3.7% 说明这个靶点对人类来说是真的难，不是随便挑的软柿子。十倍差距不是「AI 稍微好一点」，是这道题的解法变了。竞赛类基准的价值在这里体现出来了：有一条人类在同题下的真实分数线，AI 的成绩才有坐标。

---

## 5. TNFα：弱模型赢了强模型

> "We're not sure why Opus 4.8 was successful on this target and Mythos Preview was not. When we assess our models capabilities, we do so holistically."

TNFα 是个重量级靶点——阻断它是 Humira 这类史上最赚钱药物的作用原理。它难在结构是多聚体，结合位点藏在两个蛋白形成的凹槽里。

反直觉的地方：整体更强的 Mythos Preview 在这个靶点上失败了，整体更弱的 Opus 4.8 成功了，还设计出了跨物种结合剂——同时结合人、食蟹猴、小鼠的 TNFα。跨物种这件事对后面做动物实验至关重要。

Anthropic 直说了：不知道为什么。

**My take：** 这条对所有用模型的人都有用。「哪个模型更强」在排行榜上是一个标量，在真实任务上不是。任务分布够复杂时，弱模型在某些子区间赢强模型是常态，不是异常。所以选模型别只看总分——在你自己的任务上各跑一遍，成本远低于你以为的。

---

## 6. β-折叠：这是结构推理，不是模式匹配

> "Claude designed 15 confirmed binders across six targets that contain at least 20% β-strand, demonstrating its ability to reason about protein structure."

计算设计出来的结合剂，绝大多数是 α-螺旋束——一堆螺旋捆在一起。原因很简单：好设计、好折叠、不容易出错。

β-折叠不一样。它要求伸展的氨基酸链条并排对齐，更难设计，更容易错误折叠和聚集（蛋白分子粘成一团而不是各自正确折叠）。

Claude 交出了 15 个确认结合的 β-折叠设计，跨 6 个靶点，来自 10 个不同的骨架。

**My take：** 这是我认为整篇里最有信息量的一条。如果模型只是在复现训练数据里的常见模式，它会一路输出 α-螺旋束——那是文献里最多的形态。能稳定产出更难、更少见、更容易失败的结构，说明它在按物理约束推理，而不是按频率采样。判断一个模型是不是「真会」，看它敢不敢走人少的那条路。

---

## 7. 3 万 token 的提示词，连同全部数据一起开源

> "In the meantime, we are sharing the prompts we used for these campaigns, as well as all _in vitro_ and _in silico_ data we generated."

开源的东西包括三块：那份约 3 万 token 的蛋白质设计提示词、设计出的蛋白复合物的计算模型、全部体外和计算机模拟实验数据。地址是 HuggingFace 上的 `Anthropic/claude-protein-binder-design`。

3 万 token 是什么概念？大概是一本小册子。里面装的是这个领域的方法学、评判标准、失败模式、该跑哪些检查。

**My take：** 我最看重这块。命中率数字可以争论口径，但提示词和原始数据放出来之后，任何一个有 GPU 和实验预算的团队都能复现或证伪。这才是科学的做法，不是发布会的做法。

顺便，这 3 万 token 本身就是一份高价值的提示词工程样本：把一个专业领域的隐性知识写成可执行的指令，这件事在别的行业同样成立。想学怎么给专业任务写提示词的，去读它，比读十篇「提示词技巧」有用。

---

## 8. 分析化学：19 分钟 vs 4 天

> "Claude worked out how the data was encoded, then confirmed it had read the file correctly by reproducing the instrument's own recorded totals for all 2,664 scans before analyzing anything."

第二个实验用的是普通人也能用到的 Opus 5。

任务：给它一家外包实验室的原始仪器文件（NMR 核磁 + LC-MS 液质联用），加一句两句话的白话提示词，没有厂商软件，没有操作员。

结果：NMR 用了 23 分钟，LC-MS 用了 19 分钟，并行跑。每个峰的氢原子数与实验室结果差在 0.08 个 ¹H 以内，纯度测出 96.4%，实验室是 96.33%。

LC-MS 那个文件是厂商专有的、无文档的二进制格式，本来只有厂商软件能读。Claude 自己把编码方式反推出来了，然后——这是关键——在分析任何东西之前，先用 2664 次扫描复现了仪器自己记录的总量，确认自己没读错。

对照组：人工分析一个这样的样本通常要半小时到一小时，而这家实验室的成品报告是在第一张谱图采集后 4 天才送到的。

Claude 还做了一件像科学家的事：它标记出 4 个宽峰，怀疑是连在氮或氧上的氢，主动提出跑重水实验去验证——而实验室在三天后独立地做了同一个实验。拿到重水数据后，它推翻了自己第一遍的结论（第一遍说 4 个峰全消失了，自查发现只消失了 2 个），改成了和实验室操作员一致的答案。

**My take：** 破解专有格式这件事很炫，但「先复现仪器总量再开始分析」才是真正的专业素养——先验证自己的读取管线，再信自己的结论。这是很多人类分析师都会跳过的一步。至于 4 天 vs 25 分钟，那不是效率提升，那是研发节奏的相变：当反馈从「按天」变成「按分钟」，实验设计的方式本身会变。

---

## 9. 诚实：MBP 的 90 个设计全军覆没

> "Against MBP, however, none of the 90 designs was confirmed to have bound to the target, although one demonstrated a weak, reproducible binding signal."

有两个靶点 Claude 明显吃力：BBF-14 和麦芽糖结合蛋白（MBP）。

BBF-14 是个人工从头设计出来的 β-桶状蛋白，自然界不存在，正因为「新」才被当作基准。Claude 还是做出了 3 个独立的结合剂（三条设计路线各出一个，骨架各不相同），但亲和力平平。

MBP 更惨：90 个设计，0 个确认结合，只有 1 个有微弱但可重复的信号。MBP 是个大而柔软的细菌蛋白，表面光滑亲水，结合剂根本找不到能抓的地方。

**My take：** 这一节写进去了，整篇的可信度就上去了。Anthropic 甚至写了 GDF-8（成熟型）因为靶点聚集和非特异性粘附导致数据不可用，所以 16 个靶点只报了 15 个。愿意把失败案例和数据剔除理由一起写出来的技术报告，比只有漂亮数字的那种值钱得多。

---

## 10. 双用途困境：最强的能力，普通人用不到

> "As we work to deliver these capabilities safely via trusted access programs, protein design and other dual-use research biology capabilities remain unavailable for general access in Claude Fable 5."

能自主做生物研究的 AI 是双刃的。同一套能力，可以加速药物研发，也可以帮坏人做危险研究。

所以 Anthropic 的处理是：生命科学研究任务在他们最强的模型里被封锁，蛋白质设计这类能力不对普通用户开放，只通过受信访问计划交付。他们说，最高优先级之一是尽快推出面向科学家的访问计划。目前普通人能用到的最强模型是 Opus 5。

最后是结论部分的自我降温：

> "Protein minibinders are not a standard therapeutic modality for drugs and even for the common drug modalities, such as monoclonal antibodies and small molecules, designing a high-affinity binder is just the first step in the process of generating a drug-like molecule."

minibinder 不是标准药物形态。就算是抗体和小分子这些主流形态，设计出一个高亲和力结合剂也只是第一步——后面还有毒性、药代、临床试验这些真正的硬骨头。

**My take：** 两件事同时成立：这是一个货真价实的技术突破；这离「AI 造出一款药」还差着整条产业链。愿意在自己的成绩单最后一段亲手写下第二句的公司不多。

至于双用途那部分——按能力分层发放、按可信度分级访问，大概率会成为前沿模型在高风险领域的默认形态。这意味着「最强的模型」和「你能用到的最强模型」从此是两个东西，而且差距只会拉大。

---

## 写在最后

把这两个实验放在一起看，主线是同一条：**AI 正在进入「验证很贵」的领域**。

数学、代码这类地方 AI 跑得快，是因为验证便宜——错了立刻知道。生命科学是反过来的：一次验证要几周、要真金白银、要一整间实验室。在这种地方，AI 的价值不是「多试几次」，而是把每一次昂贵验证的命中率提上去。

从 10% 到 35%，翻的不是准确率，是研发预算的效率。

对做工具、做产品的人，还有一层更直接的启示：Claude 在这里不是取代了那些专业模型，它是把它们编排起来了。真正被替掉的，是那个坐在中间、决定「先跑哪个、再跑哪个、这轮结果不行要不要重来」的角色。

这个角色在每个行业都有。

---

*资料来源：Anthropic 研究博客 How Claude is accelerating protein design and analytical chemistry*
*开源数据与提示词：https://huggingface.co/datasets/Anthropic/claude-protein-binder-design*
