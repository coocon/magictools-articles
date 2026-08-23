# Claude Code Plan Mode 实战：先规划后动手，把返工率打下来

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-plan-mode-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-plan-mode-guide?utm_source=github&utm_medium=referral)**

用 Claude Code 最贵的不是 token，是返工：它自信满满地改了八个文件，你看完发现方向就错了，只能全部回滚重来。Plan Mode 就是针对这个问题的开关——先让 Claude 只读调研、给出方案，你点头之后它才碰代码。

## Plan Mode 是什么：一个"只读+审批"的工作模式

进入 Plan Mode 后，Claude 的行为会变成两段式：

1. **调研阶段（只读）**：可以读文件、搜代码、分析结构，但不能编辑文件、不能执行会改变系统状态的命令
2. **审批阶段**：把调研结论整理成实施计划呈给你；你批准后它退出 Plan Mode 开始动手，你不满意就继续讨论修改方案

关键价值在于把"想清楚"和"干活"在机制上分开了。没有 Plan Mode 时你也可以在提示词里写"先别改代码"，但那只是口头约定；Plan Mode 是硬约束，调研阶段想改文件也改不了。

## 怎么进入：Shift+Tab 循环切换

会话中按 **Shift+Tab** 可以在三种权限模式间循环：

- **普通模式**：每个敏感操作都请求确认
- **自动接受（auto-accept）**：文件编辑自动放行，适合信得过的机械性任务
- **Plan Mode**：只读规划，改动一律拦截

也可以在启动时直接指定：`claude --permission-mode plan`。如果你所在团队约定"大改动必须先出方案"，把这条写进项目 `CLAUDE.md`，Claude 会主动在非平凡任务上先进入规划流程。

## 什么任务该开 Plan Mode

一个实用的判断标准：**改动步数超过 3 步，或者存在多个合理方案时，就先规划**。具体来说：

**应该开：**

- 跨多文件的重构（"把认证逻辑抽成中间件"）
- 陌生代码库的首次改动——调研阶段本身就是熟悉代码的过程
- 有架构分歧的功能（"加实时通知"——WebSocket 还是轮询？）
- 修一个根因不明的 bug：先让它只读排查、给出诊断，避免"边猜边改"污染现场

...

---

**[👉 继续阅读全文：Claude Code Plan Mode 实战：先规划后动手，把返工率打下来](https://tools.cooconsbit.com/zh/articles/claude-code-plan-mode-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
