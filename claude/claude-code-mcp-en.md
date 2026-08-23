# Claude Code MCP: Connect External Tools and Data Sources

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-mcp-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-mcp-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Claude Code MCP: Connect External Tools and Data Sources](https://tools.cooconsbit.com/en/articles/claude-code-mcp-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
