---
title: "Claude Code MCP：连接外部工具和数据源"
slug: "claude-code-mcp"
category: "claude"
tags: [claude-code, mcp, integration, advanced]
summary: "了解 Model Context Protocol (MCP) 如何让 Claude Code 连接数据库、API 和第三方服务，扩展 AI 编程助手的能力边界。"
status: "published"
---

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

配置完成后重启 Claude Code，即可在对话中使用这些外部工具。

## 常用 MCP 服务器

| 服务器 | 功能 | 包名 |
|--------|------|------|
| GitHub | 管理 Issue、PR、仓库 | `@modelcontextprotocol/server-github` |
| Filesystem | 安全的文件系统访问 | `@modelcontextprotocol/server-filesystem` |
| SQLite | 查询 SQLite 数据库 | `@modelcontextprotocol/server-sqlite` |
| PostgreSQL | 查询 PostgreSQL 数据库 | `@modelcontextprotocol/server-postgres` |
| Slack | 发送消息、管理频道 | `@modelcontextprotocol/server-slack` |
| Memory | 跨会话持久化记忆 | `@modelcontextprotocol/server-memory` |

更多社区开发的 MCP 服务器可以在 [MCP Servers 仓库](https://github.com/modelcontextprotocol/servers) 中找到。

## 编写简单的 MCP 服务器

你可以用 TypeScript 快速编写自定义 MCP 服务器：

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({ name: "my-server", version: "1.0.0" });

server.tool(
  "get_weather",
  "获取指定城市的天气信息",
  { city: z.string().describe("城市名称") },
  async ({ city }) => {
    // 调用天气 API
    const data = await fetch(`https://api.weather.com/${city}`);
    return { content: [{ type: "text", text: JSON.stringify(await data.json()) }] };
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

## 安全注意事项

- **最小权限原则**：只授予 MCP 服务器必要的权限和访问范围
- **敏感信息保护**：API 密钥通过 `env` 字段传递，不要硬编码在配置中
- **网络隔离**：对于访问内部服务的 MCP 服务器，确保网络访问控制得当
- **审查第三方服务器**：使用社区 MCP 服务器前，务必审查其源代码

## 常见问题

### MCP 服务器需要一直运行吗？

不需要。Claude Code 会在启动时自动拉起配置的 MCP 服务器进程，会话结束时自动关闭。你不需要手动管理服务器的生命周期。

### 一个项目可以同时使用多个 MCP 服务器吗？

可以。你可以在 `mcpServers` 中配置任意数量的服务器，Claude 会根据任务需要自动选择合适的工具调用。不同服务器的工具会合并到 Claude 的可用工具列表中。

### 自定义 MCP 服务器支持哪些编程语言？

MCP SDK 官方提供了 TypeScript 和 Python 版本。社区也有 Rust、Go、Java 等语言的实现。只要遵循 MCP 协议规范，任何语言都可以编写 MCP 服务器。
