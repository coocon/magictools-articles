# AI 发现弱点用了 60 小时，人类验证用了一个月：HAWK 成为第一个被 AI 淘汰的 NIST 候选算法

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/hawk-first-algorithm-retired-by-ai?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/hawk-first-algorithm-retired-by-ai?utm_source=github&utm_medium=referral)**

这件事的完整弧线只有 48 小时。

7 月 28 日，Anthropic [发布研究](https://www.anthropic.com/research/discovering-cryptographic-weaknesses)：其内部未发布的模型 Claude Mythos Preview，半自主工作约 60 小时、消耗约 10 万美元的算力，把 NIST 后量子签名候选算法 HAWK 的已知最优分析从 2^64 的工作量压到了 2^38——等效安全强度直接砍半，HAWK-256 的密钥在单台服务器上几小时内可恢复。

7 月 29 日，HAWK 团队在 [NIST 官方邮件列表](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/2r2u6SbHun4)确认攻击有效，**正式从 NIST 标准化流程撤回算法**。

一个经过两轮、历时两年专家评审的候选算法，从「被 AI 发现弱点」到「主动退赛」，用时不到两天。这是密码学标准化历史上第一次：一个算法的出局，第一作者是 AI。

值得写的不是「AI 破解密码」这个耸动标题——独立密码学家的第一反应恰恰是让大家别慌。值得写的是藏在时间线里的另一组数字：**AI 发现这个弱点用了 60 小时，Anthropic 的人类研究员验证它用了将近一个月。** 这个比例失衡，才是这条新闻里真正的新东西。

## 先看清楚被打掉的是什么

HAWK 需要一点背景。NIST 主线的后量子标准已经定稿——ML-KEM（FIPS 203）、ML-DSA（FIPS 204）、SLH-DSA（FIPS 205），你的 TLS 连接正在逐步迁移过去的就是它们，**这次事件不影响其中任何一个**。HAWK 参加的是 NIST 的「补充签名算法」竞赛，2026 年 5 月刚进入第三轮，9 个候选里它是**唯一的格基（lattice-based）方案**——这个身份很重要，后面会用到。

攻击本身的数学面目，NIST 论坛帖的标题概括得很精确：「HAWK-n 的密钥恢复可以归约到 n/2+1 维的 SVP 问题」。用人话说：Mythos 找到了一条路，把恢复密钥所需的格基归约计算的规模砍掉了一半。约翰霍普金斯的密码学家 Matthew Green [在事后分析](https://blog.cryptographyengineering.com/2026/07/29/some-notes-about-anthropics-new-results/)里点出了一个容易被忽略的细节：**Mythos 没有发明任何新数学**，它做的是把几个已知的密码分析工具用一种没人试过的方式组合起来，攻击 HAWK 的数学根基（格同构问题）。

这里有个几乎所有报道都没展开的关键转折：**Anthropic 攻击的只是 HAWK-256，而 HAWK 提交给 NIST 的是 512 和 1024 位版本。** 按常理，团队完全可以辩护「参数够大就没事」。那为什么第二天就全面撤回？

...

---

**[👉 继续阅读全文：AI 发现弱点用了 60 小时，人类验证用了一个月：HAWK 成为第一个被 AI 淘汰的 NIST 候选算法](https://tools.cooconsbit.com/zh/articles/hawk-first-algorithm-retired-by-ai?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
