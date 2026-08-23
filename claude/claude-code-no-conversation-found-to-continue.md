# Claude Code 报 No conversation found to continue：你的 -p 会话被交互模式过滤掉了

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-no-conversation-found-to-continue?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-no-conversation-found-to-continue?utm_source=github&utm_medium=referral)**

## 官方文档怎么说

CLI 参考里 `--continue` 的语义非常简单：**继续当前目录下最近的一次会话**。没有任何附加条件——不区分这个会话是交互模式开的，还是 `claude -p "..."` 无头模式跑出来的。

这正是很多自动化工作流的基础假设：脚本里用 `-p` 批量干活，出问题了人再用 `--continue` 进交互模式接手排查。

## 实测发生了什么

macOS + Claude Code v2.1.220，一个干净的空目录，四步实验（终端为真实 TTY）：

```console
$ claude --max-turns 1 -p 'The answer is 42. Reply only with: OK'
OK

$ claude --continue
No conversation found to continue        # ← BUG：会话明明存在

$ claude -p --continue "What did I ask earlier?"
42                                        # ← 无头续聊完全正常

$ CLAUDE_CODE_ENTRYPOINT=sdk-cli claude --continue
# ← 交互界面正常打开，且完整加载了 -p 会话的历史
```

第 2 步和第 3 步的对比是关键：**同一份会话数据，无头模式续得上，交互模式说找不到**。会话文件好端端躺在 `~/.claude/projects/<项目路径>/` 下的 `.jsonl` 里——我们检查过，`-p` 创建的会话和交互会话存储在同一个位置、同一种格式。

## 坑在哪：一次过滤逻辑的误伤

这是 v2.1.90 引入的回归。那个版本的变更说明写着：*"Changed --resume picker to no longer show sessions created by `claude -p` or SDK invocations"*——`--resume` 的会话选择器不再展示 `-p`/SDK 创建的会话，这本身是合理的产品决策（选择器里全是脚本跑的一次性会话确实很吵）。

...

---

**[👉 继续阅读全文：Claude Code 报 No conversation found to continue：你的 -p 会话被交互模式过滤掉了](https://tools.cooconsbit.com/zh/articles/claude-code-no-conversation-found-to-continue?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
