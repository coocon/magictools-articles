# 左脚踩右脚的上升飞轮，没那么容易实现

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/self-tuning-flywheel-wechat-pitfall?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/self-tuning-flywheel-wechat-pitfall?utm_source=github&utm_medium=referral)**

## 想法很美

「码农早餐」是我们的日更技术公众号，每天早 8 点一篇，稿子由一条 LLM 管线生成。9 月 11 日，我给它加了一个自优化循环，设想是这样的：

1. 每天 08:50 拉公众号官方数据（阅读、推荐、转发、关注、完读）
2. 给昨天那篇所属的「写作版本」打分，和父版本比：低于 0.8 倍回滚，不低于 1 倍保留
3. LLM 读数据、提方案，在 25 个类型化旋钮里最多改 3 处，成为新版本
4. 旋钮覆盖选材门禁、标题和开头附注、篇幅、幽默密度、封面风格、主题色、关注钩子文案

判定写成代码，不交给 LLM；底线（事实校对、来源真实、不编造亲历）不在旋钮里，任何版本都碰不到。数据好 → 写得更好 → 数据更好，左脚踩右脚往上走。

40 天后回头看，这个飞轮一圈都没真正转起来。

## 40 天的账

| | 数字 |
|---|---|
| 版本数 | 18（优化器提案 12，人工设置 6） |
| 被回滚 | 8 个，其中 3 个是人工版本 |
| 循环上线后的文章 | 47 篇，合计 2818 阅读、44 次转发、7 个文内关注 |
| 阅读 ≤ 50 的文章 | 28 篇（六成） |
| 转发集中度 | 44 次转发里 21 次来自两篇（eBPF 开销砍 90%、LiteLLM 换 Rust 内核） |

三个细节比总数更说明问题：

- **v10 的旋钮和父版本 v6 一模一样**（那天优化器没提出有效方案，直接沿用父版本），照样被判回滚。这就是一次意外的 A/A 测试：同一套配置，换两天的文章，分数差到了回滚线以下。
- **封面风格在绕圈**：dark → light（v3）→ dark（v5）→ flat（v9，回滚）→ light（v14，回滚）。首屏定位行在 v8、v11 两次被提出、两次被回滚，最后是人在 v17 定的。
- **「停掉硬菜」这个决定，人设了三次**：09-16 看完 10 天数据拍板停掉千字长段（v4），09-18 优化器拿 1 篇的分数把它回滚了——那篇最终 387 阅读，是循环上线以来最高的一篇；09-19 再设（v7），又被回滚；直到 10-12 的 v17 第三次设。

...

---

**[👉 继续阅读全文：左脚踩右脚的上升飞轮，没那么容易实现](https://tools.cooconsbit.com/zh/articles/self-tuning-flywheel-wechat-pitfall?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
