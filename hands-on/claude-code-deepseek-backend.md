# 拿 DeepSeek 当 Claude Code 后端：功能全通，但成本显示虚高 38 倍

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-deepseek-backend?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-deepseek-backend?utm_source=github&utm_medium=referral)**

DeepSeek 的 API [支持 Anthropic 格式](https://api-docs.deepseek.com/guides/anthropic_api)，端点是 `https://api.deepseek.com/anthropic`。三个环境变量，Claude Code 就转去调 DeepSeek 了。

教你设这三个变量的文章有的是，讲**什么会退化**的几乎没有。而这才是唯一值得问的问题——Agent 编程工具不是聊天框，它的命脉在工具调用、多轮状态，以及屏幕上那些数字到底有没有意义。

所以我跑了一遍。Claude Code **2.1.270**，macOS，独立配置目录不碰真实环境。下文 Claude Code 的跑测全部打向 `deepseek-v4-pro`；`deepseek-flash` 出现在裸端点的映射测试和 Haiku 槽位里。

先给结论：**功能没问题，成本显示有问题。** Claude Code 告诉我花了 $1.71，DeepSeek 实际扣了 ¥0.32。

## 背景：为什么有人要这么接

两个原因，方向还不太一样。

无聊的那个是价格。DeepSeek V4 Pro 的输入价是**峰时 $1.32 / 1M token、谷时 $0.66**，和 Claude 的百万单价不在一个档。而 Claude Code 每一轮都要重发一大坨系统提示词，这个比例会被放得很大。

有意思的那个是：Claude Code 目前是被广泛使用的 Agent 框架里最能打的一个，而框架恰恰是你没法轻易自己造的部分。**保留框架、换掉底下的引擎**，如果换得干净，价值很实在。

换得干不干净是个实证问题，所以有了下面的测量。

## 测了什么，以及故意没测什么

**测了**：连通性、模型名映射、本地工具矩阵、子代理派生、跨轮 prompt 缓存，以及按真实账户余额算的实际花费。

**没测（以及为什么）**：

- **MCP 服务器**。DeepSeek 的兼容性表里把 API 的 `mcp_servers` 字段标为 *Ignored*，但那个字段指的是 Anthropic 的**服务端** MCP。Claude Code 的 MCP 走本地 stdio，根本不碰这个字段。测 API 字段证明不了你实际在用的东西，而认真测本地 MCP 够单开一篇。
- **1M 长上下文行为**。DeepSeek 标称 1M 上下文，Claude Code 报告该模型 `contextWindow: 200000`。到底谁说了算需要专门的填充实验，我不打算靠一个元数据字段猜。
- **视觉**。价格页写明 v4-pro **不支持** vision、flash 支持。我没测图片输入。

以下全部在同一台机器、2026-09-20 当天跑出，原始日志留存。

## 配置

起作用的就三个变量：

```bash
export ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic"
export ANTHROPIC_AUTH_TOKEN="$DEEPSEEK_API_KEY"   # 不是 ANTHROPIC_API_KEY
unset ANTHROPIC_API_KEY
export ANTHROPIC_MODEL="deepseek-v4-pro"
```

要用 `ANTHROPIC_AUTH_TOKEN` 而不是 `ANTHROPIC_API_KEY`——两个都设 Claude Code 会弹冲突提示。我还把整个实验塞进一次性的 `CLAUDE_CONFIG_DIR` 和临时工作目录，保证碰不到真实项目：

```bash
export CLAUDE_CONFIG_DIR="$PWD/cc-config"
```

## 先建 ground truth：这个端点到底怎么行为

动 Claude Code 之前先打裸端点。映射层要是有意外，上面那层全是猜。

### 模型映射只认前缀——而且文档关于回落的说法是错的

| 传入的 model | 实际返回的 model | 耗时 |
|---|---|---|
| `claude-opus-5` | `deepseek-v4-pro` | 3.20s |
| `claude-sonnet-5` | `deepseek-flash` | 1.03s |
| `claude-haiku-4-5-20251001` | `deepseek-flash` | 1.32s |
| `deepseek-v4-pro` | `deepseek-v4-pro` | 1.97s |
| `deepseek-flash` | `deepseek-flash` | 1.44s |
| `totally-made-up-model` | **HTTP 400** | 0.38s |

...

---

**[👉 继续阅读全文：拿 DeepSeek 当 Claude Code 后端：功能全通，但成本显示虚高 38 倍](https://tools.cooconsbit.com/zh/articles/claude-code-deepseek-backend?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
