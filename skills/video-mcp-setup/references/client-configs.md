# Video MCP Client Configuration Reference

## Table of Contents
- [Authentication Modes](#authentication-modes)
- [Claude Code](#claude-code)
- [Claude Desktop & claude.ai](#claude-desktop--claudeai)
- [Cursor](#cursor)
- [VS Code (GitHub Copilot)](#vs-code-github-copilot)
- [OpenAI Codex CLI](#openai-codex-cli)
- [Gemini CLI](#gemini-cli)
- [Windsurf](#windsurf)
- [Agent Frameworks (OpenAI Agents SDK, LangChain, Mastra, n8n, Antigravity)](#agent-frameworks)

## Authentication Modes

1. **OAuth 2.1 with PKCE (recommended for interactive clients)**:
   - URL: `https://mcp.ceptory.com/mcp`
   - Discovery flow: `401` → `WWW-Authenticate` `resource_metadata` → `https://www.ceptory.com/.well-known/oauth-authorization-server` (dynamic client registration).
   - Opens the sign-in page in the browser to approve; sessions expire after 30 days and can be revoked anytime at `https://www.ceptory.com/dashboard/mcp`.
2. **Bearer Token (headless / CI / stdio)**:
   - Create a token at `https://www.ceptory.com/dashboard/mcp`.
   - Send `Authorization: Bearer <token>` over Streamable HTTP, or pass `CEPTORY_TOKEN=<token>` to `npx -y ceptory-mcp`.
   - The stdio bridge `ceptory-mcp` adds two local-file helpers (`upload_video_file` and `image_path`) that remote Streamable HTTP clients cannot access directly.

## Claude Code
```bash
# OAuth 2.1 (then run /mcp in Claude Code -> Authenticate)
claude mcp add --transport http video-intel https://mcp.ceptory.com/mcp

# Bearer token
claude mcp add --transport http video-intel https://mcp.ceptory.com/mcp \
  --header "Authorization: Bearer $CEPTORY_TOKEN"
```

## Claude Desktop & claude.ai
- **Connectors UI (OAuth)**: `Settings → Connectors → Add custom connector` → Name: `Video Intelligence`, URL: `https://mcp.ceptory.com/mcp`.
- **Stdio bridge (`claude_desktop_config.json`)**:
```json
{
  "mcpServers": {
    "video-intel": {
      "command": "npx",
      "args": ["-y", "ceptory-mcp"],
      "env": { "CEPTORY_TOKEN": "<token>" }
    }
  }
}
```

## Cursor
File: `~/.cursor/mcp.json` or `.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "video-intel": {
      "url": "https://mcp.ceptory.com/mcp"
    }
  }
}
```
For Bearer token auth, add `"headers": { "Authorization": "Bearer <token>" }`.

## VS Code (GitHub Copilot)
File: `.vscode/mcp.json` (or Command Palette → `MCP: Add Server`):
```json
{
  "servers": {
    "video-intel": {
      "type": "http",
      "url": "https://mcp.ceptory.com/mcp"
    }
  }
}
```

## OpenAI Codex CLI
```bash
codex mcp add video-intel --url https://mcp.ceptory.com/mcp
codex mcp login video-intel
```
Or token via `~/.codex/config.toml`:
```toml
[mcp_servers.video-intel]
command = "npx"
args = ["-y", "ceptory-mcp"]
env = { CEPTORY_TOKEN = "<token>" }
```

## Gemini CLI
File: `~/.gemini/settings.json`, then run `/mcp auth video-intel`:
```json
{
  "mcpServers": {
    "video-intel": {
      "httpUrl": "https://mcp.ceptory.com/mcp"
    }
  }
}
```

## Windsurf
File: `~/.codeium/windsurf/mcp_config.json`:
```json
{
  "mcpServers": {
    "video-intel": {
      "serverUrl": "https://mcp.ceptory.com/mcp"
    }
  }
}
```

## Agent Frameworks
For OpenAI Agents SDK, LangChain, Mastra, n8n, or Antigravity:
- **Streamable HTTP**: `https://mcp.ceptory.com/mcp` with `Authorization: Bearer <token>` (MCP `2026-07-28`; 2025-era clients supported).
- **Stdio**: command `npx`, args `["-y", "ceptory-mcp"]`, env `CEPTORY_TOKEN=<token>`.
- Machine-readable server documentation: `https://www.ceptory.com/mcp/llms.txt`.
