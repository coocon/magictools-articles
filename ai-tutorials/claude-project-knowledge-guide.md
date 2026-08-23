# Claude 项目知识库指南：复用上下文，不用每次重复

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-project-knowledge-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-project-knowledge-guide?utm_source=github&utm_medium=referral)**

Claude Projects 适合处理反复出现的工作。与其在每次新对话里重新贴背景材料，不如把文档、上下文和规则放进同一个项目里，让 Claude 围绕这个工作流保持专注。

核心思路很简单：项目知识库负责稳定的参考材料，项目指令负责稳定的行为规则。只要把这两层分开，Claude 就能保持一致，同时不会被无关信息淹没。

## 项目知识库是做什么的

项目知识库保存的是 Claude 可以在同一项目内跨对话使用的资料。Anthropic 把 Projects 描述为有自己聊天历史和知识库的独立工作区。实际使用时，你可以一次性加入文档、笔记、代码片段或参考资料，然后在后续对话中继续复用。

它适合放这些内容：

- 反复被引用的产品说明
- 一整套研究笔记和来源材料
- 支持手册或内部 FAQ
- 需要保持一致风格和约束的客户项目

如果某些信息会在多个对话里反复用到，它通常就该放在项目知识库里。

## 项目指令是做什么的

项目指令不是资料，而是行为规则。

当你希望 Claude：

- 使用固定语气
- 从特定角色或视角回答
- 每次都遵循同样的格式
- 保持项目级别的约束或优先级

就应该把这些规则写进项目指令。

Anthropic 的帮助中心说明，项目指令会应用到项目内的所有对话。也就是说，它适合放可复用的行为规则，而不是一次性的任务细节。

## 一个清晰的项目结构

比较实用的结构是这样：

1. 把背景材料放进项目知识库。
2. 把重复出现的行为规则放进项目指令。
3. 每次新聊天只描述当前任务，不要把整个项目历史重复一遍。
4. 只有真正会复用的资料，才加入项目。

这样做很重要，因为它能减少提示词漂移。如果每次对话都把大量内容重新贴进去，Claude 就要在噪音里找重点。把稳定上下文放在项目里，每次聊天就能更短、更聚焦。

## 什么时候用项目知识库，而不是普通聊天上下文

...

---

**[👉 继续阅读全文：Claude 项目知识库指南：复用上下文，不用每次重复](https://tools.cooconsbit.com/zh/articles/claude-project-knowledge-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
