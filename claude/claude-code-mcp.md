# Claude Code MCP：连接外部工具和数据源

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-mcp?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-mcp?utm_source=github&utm_medium=referral)**

## 什么是 MCP

Model Context Protocol (MCP) 是 Anthropic 推出的开放协议，定义了 AI 模型与外部工具之间的标准通信方式。通过 MCP，Claude Code 可以连接数据库、调用 API、操作第三方服务，将 AI 编程助手的能力从代码编辑扩展到整个开发工作流。

MCP 采用**客户端-服务器架构**：Claude Code 作为 MCP 客户端，通过标准协议与 MCP 服务器通信。每个 MCP 服务器暴露一组工具（tools），Claude 可以根据需要主动调用这些工具。

## 配置 MCP 服务器

在 `.claude/settings.json` 中添加 MCP 服务器配置：

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_your_token_here"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/dir"]
    },
    "sqlite": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sqlite", "--db-path", "./data.db"]
    }
  }
}
```

...

---

**[👉 继续阅读全文：Claude Code MCP：连接外部工具和数据源](https://tools.cooconsbit.com/zh/articles/claude-code-mcp?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
