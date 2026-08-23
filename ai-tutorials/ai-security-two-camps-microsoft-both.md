# 7 月 27 日，AI 安全分裂成两个阵营——微软两边都押了注

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/ai-security-two-camps-microsoft-both?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/ai-security-two-camps-microsoft-both?utm_source=github&utm_medium=referral)**

2026 年 7 月 27 日，同一个问题收到了两份针锋相对的答案。

问题是：**AI 时代的安全能力，应该握在谁手里？**

第一份答案来自旧金山。微软在发布会上拿出了 [MAI-Cyber-1-Flash](https://microsoft.ai/news/introducing-mai-cyber-1-flash-inside-mdash/)——它的第一个网络安全专用自研模型，闭源、私有预览、只对通过审核的客户开放。微软 AI 负责人 Mustafa Suleyman 的配套表态毫不遮掩：数据、harness、专业知识，这是护城河，而且「这只是冰山一角」。

第二份答案来自 Nvidia 的官方博客。[Open Secure AI Alliance](https://blogs.nvidia.com/blog/open-secure-ai-alliance/) 宣告成立，37 家创始成员（不同报道给出 37 到 52 家不等的口径，以 Nvidia 官方名单为准），包括 IBM、CrowdStrike、Palo Alto Networks、Cloudflare、Hugging Face、Linux Foundation。联盟的核心主张一句话就能说完：**防御者需要能审计、能修改、能本地运行的安全 AI**——言外之意，闭源 API 做不到。

有意思的地方在名单里：**微软两边都在。** 它是联盟创始成员，同一天发布了一个恰好违背联盟核心主张的闭源模型。

这不是编辑巧合，是行业在一天之内把底牌全摊开了。这篇文章把两边的逻辑分别拆开，再看微软的双重下注到底是精神分裂还是精明分层。

## 微软的答案：安全能力是护城河

先看清楚微软发布的到底是什么，因为转述报道里的信息损耗不小。

MAI-Cyber-1-Flash 基于微软自研的 MAI-Thinking-1 推理模型，被设计为在 **MDASH** 里干活——MDASH 是微软 5 月发布的多智能体漏洞识别与修复编排系统，由安全专家调教的 100 多个 agent 组成。分工是明确的：MAI-Cyber-1-Flash 处理最多 90% 的任务，剩下最难的 10% 交给 GPT-5.4。微软宣称这套组合的成本，比之前「GPT-5.4 + GPT-5.4 mini + GPT-5.3 Codex」的配置**低 50%**。

成绩单：在 [CyberGym](https://thehackernews.com/2026/07/microsoft-says-new-cybersecurity-ai.html)（1507 个真实漏洞复现任务，取自 OSS-Fuzz 覆盖的 188 个开源项目）上拿到 **95.95%**，领先 Anthropic 的 Mythos 系统约 12 个百分点，也高于 GPT-5.5-Cyber 的约 83%。

产品化路径：通过 **Project Perception** 落地——红队 agent 模拟攻击路径、蓝队 agent 调查定险、绿队 agent 执行修复，**8 月 3 日进入公测**。同场还宣布了新的安全研究部门 FORGE Labs。

...

---

**[👉 继续阅读全文：7 月 27 日，AI 安全分裂成两个阵营——微软两边都押了注](https://tools.cooconsbit.com/zh/articles/ai-security-two-camps-microsoft-both?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
