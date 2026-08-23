# 使用 Claude 的首选语言：实用工作流

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-preferred-language-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-preferred-language-guide?utm_source=github&utm_medium=referral)**

Claude 可以处理多种语言，但真正想要稳定结果，最好还是把“界面语言”和“输出语言”分开处理。Anthropic 的帮助中心说明，Claude 的网页和桌面应用支持多种界面语言，同时即使界面语言不同，Claude 也可以继续用你输入的语言进行对话。

这个区别很重要。界面语言影响的是应用体验，提示词语言影响的是回答内容。如果你要做双语工作流，就应该把这两件事分开指定。

## 先设置界面语言

如果你希望 Claude 应用本身也更符合本地使用习惯，可以先到账号设置里修改界面语言。Anthropic 的帮助中心列出的支持语言包括英语、法语、德语、印地语、印尼语、意大利语、日语、韩语、葡萄牙语以及西班牙语变体。

调整界面语言适合这些场景：

- 团队更习惯非英语界面
- 需要给非英语使用者做引导
- 希望菜单和设置更贴合使用者习惯

但要注意，改了界面语言，并不等于 Claude 的回答语言也会自动跟着变。

## 在提示词里明确输出语言

如果你想让 Claude 用中文、英文、日语或其他语言回答，最好直接在提示词里说清楚。不要只依赖界面语言。

例如：

```text
请把这段会议纪要总结成英文，即使原文是中文也一样。
```

```text
请用中文回答，并保留英文产品名，不要翻译。
```

```text
Translate this draft into Spanish for a customer-facing email. Keep the tone polite and concise.
```

语言要求越重要，提示得就越明确。

## 把 Claude 当作双语工作流工具

Claude 特别适合“一个语言是源语言，另一个语言是交付语言”的场景。比较实用的做法包括：

- 先翻译客户回复，再发送
- 先用英文写技术文档，再本地化成中文
- 把访谈记录或笔记总结成团队实际使用的语言
- 将混合语言草稿统一改写成一致的最终版本

...

---

**[👉 继续阅读全文：使用 Claude 的首选语言：实用工作流](https://tools.cooconsbit.com/zh/articles/claude-preferred-language-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
