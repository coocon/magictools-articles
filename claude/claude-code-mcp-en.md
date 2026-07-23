---
title: "Claude Code MCP: Connect External Tools and Data Sources"
slug: "claude-code-mcp-en"
category: "claude"
tags: [claude-code, mcp, integration, advanced]
summary: "Learn how Model Context Protocol (MCP) lets Claude Code connect to databases, APIs, and third-party services to extend your AI coding assistant's capabilities."
status: "published"
---

## What Is MCP

Model Context Protocol (MCP) is an open protocol by Anthropic that defines a standard communication layer between AI models and external tools. Through MCP, Claude Code can connect to databases, call APIs, and interact with third-party services, extending its capabilities far beyond code editing.

MCP uses a **client-server architecture**: Claude Code acts as the MCP client, communicating with MCP servers via the standard protocol. Each MCP server exposes a set of tools that Claude can invoke on demand.

## Configuring MCP Servers

Add MCP server configurations to `.claude/settings.json`:

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

Restart Claude Code after adding configurations, and the external tools become available in your conversations.

## Popular MCP Servers

| Server | Capabilities | Package |
|--------|-------------|---------|
| GitHub | Manage issues, PRs, repos | `@modelcontextprotocol/server-github` |
| Filesystem | Secure file system access | `@modelcontextprotocol/server-filesystem` |
| SQLite | Query SQLite databases | `@modelcontextprotocol/server-sqlite` |
| PostgreSQL | Query PostgreSQL databases | `@modelcontextprotocol/server-postgres` |
| Slack | Send messages, manage channels | `@modelcontextprotocol/server-slack` |
| Memory | Persistent cross-session memory | `@modelcontextprotocol/server-memory` |

Find more community-built MCP servers in the [MCP Servers repository](https://github.com/modelcontextprotocol/servers).

## Writing a Simple MCP Server

Build a custom MCP server in TypeScript:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({ name: "my-server", version: "1.0.0" });

server.tool(
  "get_weather",
  "Get weather information for a city",
  { city: z.string().describe("City name") },
  async ({ city }) => {
    const data = await fetch(`https://api.weather.com/${city}`);
    return { content: [{ type: "text", text: JSON.stringify(await data.json()) }] };
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

## Security Considerations

- **Principle of least privilege**: Only grant MCP servers the permissions they actually need
- **Protect secrets**: Pass API keys via the `env` field; never hardcode them in configuration
- **Network isolation**: For MCP servers accessing internal services, ensure proper network access controls
- **Audit third-party servers**: Review source code of community MCP servers before using them

## FAQ

### Do MCP servers need to run continuously?

No. Claude Code automatically starts configured MCP server processes on launch and shuts them down when the session ends. You don't need to manage server lifecycles manually.

### Can I use multiple MCP servers in one project?

Yes. Configure as many servers as needed in `mcpServers`. Claude automatically selects the appropriate tool for each task. Tools from all servers are merged into Claude's available tool list.

### What programming languages can I use to write MCP servers?

The official MCP SDK is available in TypeScript and Python. Community implementations exist for Rust, Go, Java, and more. Any language can be used as long as it follows the MCP protocol specification.
