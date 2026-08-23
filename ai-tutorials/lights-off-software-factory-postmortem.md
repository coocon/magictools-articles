# 「关灯」软件工厂跑了四个月，他的联合创始人手写了两周代码善后

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/lights-off-software-factory-postmortem?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/lights-off-software-factory-postmortem?utm_source=github&utm_medium=referral)**

7 月下旬，一篇标题为《Why Software Factories Fail》（为什么软件工厂会失败）的长文冲上 Hacker News 首页，截至写稿时拿到 **390 分**，评论区吵了几百楼。它基于作者在 AI Engineer World's Fair 2026 上的主题演讲，副标题更扎心：**harness engineering is not enough**——光靠工程手段兜底，是不够的。

这篇文章值得认真读，原因和大多数「AI 写代码不行」的文章恰恰相反：

**作者不是 AI 怀疑论者，他是靠教人用 coding agent 出名的人。**

Dex Horthy，HumanLayer 创始人，《12-Factor Agents》的作者，讲「Advanced Context Engineering」的那几场演讲在 YouTube 上累计约百万播放。过去一年里，无数团队照着他的方法论搭 agent 工作流。

现在，这个人站出来说：我把「没人看代码」的软件工厂在自己公司真跑了一遍，结局是联合创始人在 VS Code 里**手写了整整两周代码**收拾残局。

## 一、先讲清楚：什么是「关灯工厂」

「软件工厂」这个词不是 2026 年发明的，它可以追溯到 1968 年——和「软件工程」出自同一场 NATO 会议。半个多世纪以来，人们一直梦想把软件生产变成流水线，而不是手艺人的孤独劳作。

AI 时代的版本大概长这样：需求进队列，agent 领任务写代码，自动化测试和 review bot 把关，出问题的事故单也自动流回队列。人的工作只剩两个问题：**往队列里塞多少活，以及多快能审完产出**。

然后有人想：既然审代码是瓶颈，把这一步删掉不就行了？

这就是「关灯工厂」（lights-off factory）。这个词借自制造业——日本 FANUC 从 2001 年起就运行不需要开灯的车间，因为车间里只有机器人，而机器人不需要光。软件版的「关灯」是：**上线的代码，没有任何人类读过**，只有机器验证过。

这不是纸上谈兵。StrongDM 公开运营着一个「无人写代码、无人读代码」的工厂，Simon Willison 今年 2 月专门写文章介绍过；OpenAI 的 Ryan Lopopolo 也在 2 月发文讲「harness engineering」，介绍内部的软件工厂 Symphony；Ramp、Stripe、WorkOS、Brex 今年都陆续宣布 agent 生产了大约 75% 的代码。

支撑这一切的叙事只有四句话：你是瓶颈；模型已经够好了；代码是免费的；多发就完了。

## 二、他真的试了：2025 年 7 月开灯变关灯，11 月人肉重写

Dex 的团队在 2025 年 7 月全面转向关灯模式：人只读 spec 和工单，中小型任务全交给后台 agent，没人看产出的代码。

按他的复盘，剧本是这样展开的：

**第一次**，出现了一个 agent 无论如何解不掉的硬骨头 bug。深度检索、上下文精编、让 agent 用十种方式复现——都不行。最后只能咬牙钻回那个三个月没人读过的代码库里人肉排障。这期间：网站宕机，用户暴怒，而他自己「痛苦地读着当初放进系统的一坨坨 slop 代码」。

第一次他忍了，「为了速度，这点下行风险值得」。

**到 11 月第三次**发生同样的事，团队做了一个决定：与其继续修，不如推倒重写。他的联合创始人在 VS Code 里——他特意强调「甚至不是 Cursor」——花了**整整两周**，把所有代码模式一行行手工重建。

从 7 月到 11 月，这个关灯实验跑了大约四个月。这个时间点值得记住，因为它和另一个观察对上了：Dex 说，以现在的出货速度，一个 agent 生成的代码库大约 **3 到 6 个月**就会进入「brownfield」状态——过去这个词指十年陈酿的 Java 老系统，现在指你自己上个季度刚 vibe 出来的项目。

## 三、根因不在工程侧，在训练侧：坏设计没有惩罚

这篇文章真正的价值不是讲故事，而是往下挖了一层：**为什么模型会这样？**

Dex 的答案要从 coding 模型怎么训练说起。强化学习（RLVR）的循环大致是：让 agent 生成解题轨迹，用某个「打分器」判分，然后更新权重让高分轨迹更常出现。这个循环要跑几百万次，所以打分必须又快又准。

...

---

**[👉 继续阅读全文：「关灯」软件工厂跑了四个月，他的联合创始人手写了两周代码善后](https://tools.cooconsbit.com/zh/articles/lights-off-software-factory-postmortem?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
