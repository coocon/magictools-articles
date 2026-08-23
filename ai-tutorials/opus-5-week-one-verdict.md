# Opus 5 发布一周：数据说它更准，体感说它更烦——这其实是同一件事

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/opus-5-week-one-verdict?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/opus-5-week-one-verdict?utm_source=github&utm_medium=referral)**

Claude Opus 5 是 7 月 24 日发布的（$5/$25 每百万 token，Artificial Analysis 智能指数 61，压过 Fable 5 的 60 和 GPT-5.6 Sol 的 59）。蜜月期一周，社区进入实测定论期——而这次的定论罕见地分裂成了两半。

数据侧，[CodeRabbit 的 code review 基准](https://www.coderabbit.ai/blog/opus-5-model-review)给出一句冷静的判词：「精确率专家，适合与召回型模型搭配使用」。体感侧，ChatPRD 创始人、How I AI 播客主持 Claire Vo [的开场白](https://www.chatprd.ai/how-i-ai/my-surprising-verdict-on-claude-opus-5)是：「我*讨厌*和它共事——然而在盲测里，我把它排在了所有模型之上，包括 Fable 和我心爱的 GPT-5.6。」

「更准」和「更烦」，看起来是两条新闻。把数字摆开之后你会发现，它们是**同一件事**。

## 数据线：史上最准，同时更漏、更吵

先看 CodeRabbit 的方法论，因为它比多数厂商评测扎实：96 个错误模式全部来自真实开源 PR 的已验证 issue（不是合成 bug），每个配置跑 3 次，对照当前生产模型组合的 3 次运行。

结果是一组教科书级的 trade-off：

- **可执行评论精确率 39.3%，对比基线 35.2%**——CodeRabbit 有史以来测到的最高值
- **已知 bug 召回率 55.2%，对比基线 61.1%**——更准的同时，漏掉的真 bug 更多了
- **nitpick（低价值挑剔评论）约 92 条，对比基线 23 条**——翻了 4 倍
- 全流（含所有评论类别）精确率 **28.6%，低于基线的 32.8%**

更有意思的是档位实验：把 reasoning effort 降到默认档，发现的问题总数反而最多，但全流精确率跌到 26.4%、nitpick 冲到 110 条。CodeRabbit 的结论值得抄在本子上：**「更多推理并不稳定地产出更好的 review。选择 effort 档位，本质是在两种失败模式之间做选择。」**

...

---

**[👉 继续阅读全文：Opus 5 发布一周：数据说它更准，体感说它更烦——这其实是同一件事](https://tools.cooconsbit.com/zh/articles/opus-5-week-one-verdict?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
