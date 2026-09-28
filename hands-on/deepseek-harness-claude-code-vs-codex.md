# DeepSeek harness 实测：同一个模型换三种外壳——Claude Code 15/15、Codex CLI 15/15、裸 API 0/15 还谎报 5 次完成

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/deepseek-harness-claude-code-vs-codex?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/deepseek-harness-claude-code-vs-codex?utm_source=github&utm_medium=referral)**

搜「DeepSeek harness」的人，多半遇到过这种怪事：**同一个** DeepSeek 模型，放进这个编程工具里挺聪明，换一个工具就笨手笨脚。模型没变，变的是 **harness**——模型外面那一层：系统提示词、工具定义和命名、agent 循环、重试和超时策略，以及工具输出太大时怎么截断。

所以我把模型钉死，只换 harness。一台机器、一天（2026-09-28）、一个模型 `deepseek-v4-pro`，三种驱动方式：

| 组 | harness | DeepSeek 端点 |
|---|---|---|
| **A** | Claude Code 2.1.280 | `https://api.deepseek.com/anthropic`（Anthropic 格式） |
| **B** | Codex CLI 0.157.1（`npx @openai/codex`） | `https://api.deepseek.com`，`wire_api = "responses"` |
| **C** | 无——一次裸 `POST /chat/completions`，任务就是唯一一条 user 消息 | `https://api.deepseek.com/chat/completions` |

「harness 重要吗」的简短回答：**它决定事情有没有真的发生，决定账单差 7 倍，还决定出错时模型能看见什么。**

![同一个 DeepSeek 模型、三种 harness 的工具矩阵，每格 3 轮](https://cdn.tools.cooconsbit.com/uploads/articles/2026-09-28-deepseek-harness/02-matrix.png)

## 问题背景：为什么换 harness 而不换模型

大多数「DeepSeek vs X」的对比同时改了两样东西：模型，和模型外面的工具。这样的结果没法解读——DeepSeek 在 Codex 里比 Claude 在 Claude Code 里慢，到底是 DeepSeek 的问题还是 Codex 的问题？

DeepSeek 恰好在同一个账号下提供了**两种**适合 agent 的协议：Anthropic 兼容端点（[让 Claude Code 跑在 DeepSeek 上](/zh/hands-on/claude-code-deepseek-backend)就是靠它）和 OpenAI Responses 兼容端点。于是可以把用得最广的两个厂商 harness——Claude Code 和 Codex CLI——架在完全相同的模型、完全相同的账号上，直接量差异。

## 问题分析：把「harness」拆开

harness 这个词太虚，我把它拆成可能影响结果的几块，逐块测：

1. **工具**——有哪些工具、叫什么名字、模型用不用。
2. **提示词体量**——注入的系统提示词和工具 schema 有多大，对成本有什么影响。
3. **缓存**——这些体量是不是每次都重新全价计费。
4. **失败策略**——遇到 HTTP 500、429、连接挂起时怎么办。
5. **输出截断**——命令打印 120KB 时，模型实际看到什么。

裸 API 的 C 组是对照：同模型、同提示词、没有 harness。

## 技术方案与选型（含排除项）

**跑了什么。** 五个任务，每组每任务 3 轮，每轮在仓库之外新建空工作目录（避免任何项目说明文件被加载、污染提示词体量读数）：

- **T1-read**——回复 `probe.txt` 的内容。
- **T2-write**——新建 `written.txt`，内容为 `WRITTEN_OK`。
- **T3-edit**——把 `config.ini` 里的 `port=8080` 改成 `port=9090`，其它行不动。
- **T4-bash**——运行 `sh gen.sh`：它从 `/dev/urandom` 生成随机 nonce 写进 `nonce.txt` 并打印，要求回复这个值。读脚本猜不出来。
- **T5-multi**——读 `sales.csv`，**用 shell 命令**求和，把总数写进 `total.txt`，往 `log.txt` 追加 `verified`，回复总数（605）。

另有 **T6-bigout**：运行 `sh big.sh`（6,000 行、120,000 字符的伪随机 token），专测截断。

成败**只看磁盘**：每轮结束脚本把工作目录的 `ls -la`、`shasum -a 256`、`cat` 原样存档并据此判定，从不采信模型自己说做了什么。

调用命令原文：

```bash
# A — Claude Code，独立 CLAUDE_CONFIG_DIR
ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic \
  claude -p "<prompt>" --output-format stream-json --verbose --dangerously-skip-permissions

# B — Codex CLI，独立 CODEX_HOME
codex exec --skip-git-repo-check -C <workdir> --ephemeral --json -s workspace-write "<prompt>"

# C — 无 harness
POST /chat/completions {"model":"deepseek-v4-pro","messages":[{"role":"user","content":"<prompt>"}]}
```

两边权限设定并不对等：A 用 `--dangerously-skip-permissions`，B 用 `workspace-write` 沙箱（`exec` 模式下不弹审批）。两边都能写工作目录，这几个任务只需要这个。

...

---

**[👉 继续阅读全文：DeepSeek harness 实测：同一个模型换三种外壳——Claude Code 15/15、Codex CLI 15/15、裸 API 0/15 还谎报 5 次完成](https://tools.cooconsbit.com/zh/articles/deepseek-harness-claude-code-vs-codex?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
