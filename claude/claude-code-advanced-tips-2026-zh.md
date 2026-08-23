# Claude Code 进阶实战：会话恢复、Checkpoint 回滚与 Headless CI 的 10 个官方技巧

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-advanced-tips-2026-zh?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-advanced-tips-2026-zh?utm_source=github&utm_medium=referral)**

会用 Claude Code 和用好 Claude Code 之间，隔着一批"官方文档里写了、但多数人从没打开过的页面"。这篇不讲入门（新手请看站内的 Claude Code 快速上手指南），只挑 10 个能直接改变工作流的进阶技巧，全部来自官方文档，每条按"怎么操作 → 什么时候用"的结构展开。

## 目录

1. 给会话起名字，随时找回来
2. /rewind：代码和对话分开回滚的时光机
3. 上下文瘦身三件套：/clear、/compact、/context
4. 按路径生效的规则文件（Path-Scoped Rules）
5. 自动记忆：让 Claude 自己记笔记
6. Headless 模式：把 Claude Code 接进脚本和 CI
7. 结构化输出与成本核算
8. 探索 → 规划 → 实现 → 提交的四段式工作流
9. 富媒体上下文：截图、@文件、管道喂日志
10. Skills：把重复流程固化成一条命令

## 1. 给会话起名字，随时找回来

多任务并行时最大的痛苦是"昨天那个改到一半的会话去哪了"。解法是从一开始就命名：

```bash
claude -n payment-refactor     # 启动时命名
/rename payment-refactor       # 会话中途改名
claude --continue              # 恢复最近一个会话
claude --resume                # 打开会话选择器，按名字挑
```

配合 `/branch <name>` 还能从当前对话分叉出一个副本——主线继续改代码，分支去试另一个方案，两边共享此前的全部上下文。

**适用场景**：任何超过一天的任务、任何同时推进 2 个以上任务的人。

## 2. /rewind：代码和对话分开回滚的时光机

Claude Code 会在你每次发消息时自动打快照（checkpoint）。当它改崩了代码，你不需要手动 `git stash` 抢救：

- 按 `Esc` 两下或输入 `/rewind` 打开回滚菜单
- 选择恢复到任意一个历史时刻
- 关键是可以**只恢复代码**（对话记忆保留，Claude 还记得教训）或**只恢复对话**（代码保留，重新组织提问）

**适用场景**：让 Claude 尝试激进方案之前，你什么都不用做——快照是自动的，敢想敢试的底气就来自这里。注意 checkpoint 不能替代 git：它是会话内的撤销键，不是版本管理。

## 3. 上下文瘦身三件套：/clear、/compact、/context

长会话变笨、变贵的根源是上下文堆满了无关内容。三个命令对应三种情况：

| 命令 | 作用 | 什么时候用 |
|------|------|-----------|
| `/context` | 查看当前上下文占用明细 | 感觉响应变慢变贵时先诊断 |
| `/clear` | 清空上下文重新开始 | 切换到完全不相关的新任务 |
| `/compact 保留XX相关的决策` | 把历史压缩成摘要 | 任务没做完但上下文快满了 |

...

---

**[👉 继续阅读全文：Claude Code 进阶实战：会话恢复、Checkpoint 回滚与 Headless CI 的 10 个官方技巧](https://tools.cooconsbit.com/zh/articles/claude-code-advanced-tips-2026-zh?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
