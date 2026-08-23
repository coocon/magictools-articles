# Claude 角色提示词指南：什么时候该用 System Prompt

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-role-system-prompts-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-role-system-prompts-guide?utm_source=github&utm_medium=referral)**

在 Anthropic 的官方文档里，角色提示词是 system prompt 的一个高价值用法。它的核心逻辑并不复杂：先通过 `system` 参数告诉 Claude“你是谁、应该以什么视角工作”，再用用户消息描述当前这一轮任务。

很多用户会把角色、任务、格式要求、背景资料全部写进同一个提示词里。这样当然也能工作，但结构会越来越乱，后续维护和复用都很困难。Anthropic 官方建议把长期稳定的行为要求放到 system prompt，把会变化的任务要求留给 user prompt。

## system prompt 适合放什么

一个好的 system prompt，不是把所有项目背景一次性塞进去，而是定义那些**跨任务稳定存在**的行为规则，例如：

- Claude 要扮演什么角色
- 需要用什么专业视角思考
- 保持什么写作风格或判断框架
- 哪些边界条件需要持续生效

Anthropic 的建议非常明确：如果某条要求在多个任务之间都成立，它更适合进入 system prompt。

## 哪些场景特别适合角色提示词

角色提示词通常在下面这些场景里最有价值：

1. 你需要更符合某个专业领域的分析方式。
2. 你希望多轮输出始终保持一致风格。
3. 你希望 Claude 始终站在固定的业务视角上工作。

例如，“你是一名高级安全工程师，正在审查上线方案”通常会比“帮我看看这个上线方案”更容易得到聚焦、专业的反馈。

## 一个更清晰的结构写法

下面是一个符合 Anthropic 官方建议的写法：

### System prompt

```text
你是一名资深产品运营经理，擅长为跨部门管理层撰写简洁、可决策的状态更新。你会优先指出阻塞项、依赖关系和需要拍板的事项。
```

### User prompt

```text
请根据下面的会议记录，起草一份本周项目更新。

必须包含：
1. 总体状态
2. 关键风险
3. 本周需要决策的事项

约束：控制在 220 字以内。

会议记录：
[在这里粘贴内容]
```

...

---

**[👉 继续阅读全文：Claude 角色提示词指南：什么时候该用 System Prompt](https://tools.cooconsbit.com/zh/articles/claude-role-system-prompts-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
