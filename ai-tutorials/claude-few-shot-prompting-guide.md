# Claude Few-Shot 提示词指南：用示例锁定输出格式

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-few-shot-prompting-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-few-shot-prompting-guide?utm_source=github&utm_medium=referral)**

Anthropic 在官方提示词工程文档里把“示例”视为一个非常高价值的技巧。它常被叫做 few-shot 或 multishot prompting。简单来说，就是你不要只告诉 Claude 结果应该长什么样，而是直接给它看几个高质量样例。

这招特别适合那些“你脑子里很清楚，但很难用一句话解释清楚”的任务。比如固定输出格式、分类标准、风格要求，或者边界条件复杂的判断题。相比写一大段抽象说明，几个好例子往往更有效。

## 哪些场景最适合 few-shot

示例驱动的提示词在下面几类任务里尤其有用：

- 你需要稳定一致的输出结构
- 你在做文本分类、标签判断、内容审核
- 你希望 Claude 模仿某种固定风格
- 你担心边界情况处理不一致

Anthropic 官方建议，在比较复杂的任务里可以考虑提供 3 到 5 个相关且有差异性的例子。

## 什么样的示例才算好

根据 Anthropic 的官方说明，好的示例通常有三个特点：

1. **相关**：和你真正要做的任务足够接近。
2. **多样**：能够覆盖不同常见情况，而不是只重复一种模式。
3. **清晰**：示例之间边界明显，Claude 很容易解析。

官方文档还建议使用 XML 风格标签，比如 `<example>`、`<examples>`，来显式划分示例块。这会让 Claude 更容易识别结构。

## 一个实用的 few-shot 模板

```text
请把每条客户消息分类到以下标签之一：
- 计费问题
- 技术故障
- 取消服务
- 功能建议

请参考示例中的判断方式。

<examples>
<example>
消息："我这个月被重复扣费了。"
标签：计费问题
</example>

<example>
消息："导出 PDF 时应用会崩溃。"
标签：技术故障
</example>

<example>
消息："请在当前账期结束后关闭我的账号。"
标签：取消服务
</example>
</examples>

现在请分类：
消息："[在这里插入新的客户消息]"
```

...

---

**[👉 继续阅读全文：Claude Few-Shot 提示词指南：用示例锁定输出格式](https://tools.cooconsbit.com/zh/articles/claude-few-shot-prompting-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
