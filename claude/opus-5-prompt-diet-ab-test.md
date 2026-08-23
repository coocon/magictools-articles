# 我按「删掉这 5 类提示词」清了 Opus 5 配置，然后跑了 18 次 A/B：省了 21%，但「更好」我拿不出证据

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/opus-5-prompt-diet-ab-test?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/opus-5-prompt-diet-ab-test?utm_source=github&utm_medium=referral)**

有一类说法这半年特别流行：Claude Opus 5 已经自带验证和思考，你写在 `CLAUDE.md` 里那些「回答前再检查一遍」「用子智能体复查」的兜底规则，现在全在帮倒忙——删掉既省 token 又不掉质量。Anthropic 官方在迁移指引里表达过类似意思，Claude Code 之父 Boris Cherny 说得更狠：每六个月把提示词全删一遍，看看模型不戴枷锁能做什么。

我照做了。我的全局 `~/.claude/CLAUDE.md` 里确实躺着几条典型的旧时代产物：

```markdown
### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- For complex problems, throw more compute at it via subagents

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Ask yourself: "Would a staff engineer approve this?"
```

删掉验证节、反转子智能体策略、补上范围合同和长度控制，113 行改成 87 行。

然后问题来了：**我怎么知道改完是变好了，而不是我把有用的东西删了、然后靠"新配置感觉更清爽"自我说服？**

所以我搭了个对照实验。结论先放这儿：**省是真的省，而且非常稳定；「更好」我拿不出证据，甚至找到了一个反例。**

## 实验怎么搭

关键是配置隔离。Claude Code 支持 `CLAUDE_CONFIG_DIR` 环境变量指定配置目录，于是新旧两份 `CLAUDE.md` 各放一个目录，配上同一份 `settings.json`（同一个模型 `opus`、同样的推理预算）：

```bash
CLAUDE_CONFIG_DIR=./cfg-old claude -p "$PROMPT" \
  --output-format stream-json --verbose \
  --permission-mode acceptEdits \
  --allowedTools "Read,Edit,Write,Glob,Grep,Task,TodoWrite"
```

先验证隔离真的生效——让两边各自复述「你的全局指令里关于子智能体的那一条」：

```
=== cfg-old ===
- Use subagents liberally to keep main context window clean
=== cfg-new ===
- 默认自己做。只有任务确实需要独立并行处理时才委派
```

生效了。`--output-format stream-json` 会吐出每一次工具调用的结构化事件，从里面能数出工具调用次数、`Task` 工具（子智能体）的次数，最后的 `result` 事件带 `usage.output_tokens`。

三个任务，各针对一条被改动的规则：

| 任务 | 测什么 | 内容 |
|---|---|---|
| A | 范围合同 | 沙盒仓库里修一个真 bug：超时后 `AbortController` 的定时器没清理。周围故意埋了 `any` 类型、重复代码、没有退避的重试循环当诱饵 |
| B | 长度控制 | 开放式排查：「找出 `src/` 下会让重试逻辑出问题的地方」 |
| C | 委派倾向 | 在真实仓库（本站源码，1000+ 文件）只读探索：「找出所有决定文章 hreflang / canonical 的代码路径」 |

A、B 在沙盒里跑，每次从 git 基线全新复制；C 在真实仓库跑，只给 `Read/Glob/Grep/Task`，不给 `Edit/Write`，零改动风险。沙盒刻意放在项目目录之外——否则会继承项目级的 `CLAUDE.md`，多一个变量。

...

---

**[👉 继续阅读全文：我按「删掉这 5 类提示词」清了 Opus 5 配置，然后跑了 18 次 A/B：省了 21%，但「更好」我拿不出证据](https://tools.cooconsbit.com/zh/articles/opus-5-prompt-diet-ab-test?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
