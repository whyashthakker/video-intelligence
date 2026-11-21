---
name: ceptory-mcp-setup
description: Connect AI coding and operations agents (Claude Code, Claude Desktop, Cursor, VS Code, Codex, Gemini CLI, Windsurf, or custom MCP clients) to Ceptory's MCP server (https://mcp.ceptory.com/mcp) or stdio bridge (ceptory-mcp), verify workspace account and balance, and route user requests to the right Ceptory MCP tools. Use whenever a user wants to connect an AI agent to Ceptory, authenticate with Ceptory MCP, troubleshoot 401/402/429 MCP errors, or inspect their Ceptory workspace via MCP.
---

# Ceptory MCP Server & Agent Connection

Connect any Model Context Protocol (MCP) client or agent framework to [Ceptory](https://www.ceptory.com) so the agent can manage cameras, workflows, alerts, cross-camera movement, video search, and AI detection engines with the signed-in user's workspace, role, and prepaid balance.

## Endpoints & Docs

- **Streamable HTTP endpoint**: `https://mcp.ceptory.com/mcp` (MCP `2026-07-28`, backwards-compatible with 2025-era clients)
- **Human setup docs**: `https://www.ceptory.com/mcp`
- **Machine-readable docs**: `https://www.ceptory.com/mcp/llms.txt`
- **Token management / revoke agents**: `https://www.ceptory.com/dashboard/mcp`
- **Stdio bridge (for stdio-only clients & local files)**: `CEPTORY_TOKEN=<token> npx -y ceptory-mcp` (adds `upload_video_file` and `image_path` helpers for local files)

> Note: Ceptory provides an MCP server and a REST API (`https://www.ceptory.com/api/v1`). Do not reference or invent language-specific Ceptory SDKs.

## Authentication Modes

1. **OAuth 2.1 with PKCE (recommended for interactive clients)**
   - Discovered automatically from `https://mcp.ceptory.com/mcp` (`401` → `WWW-Authenticate` `resource_metadata` → `https://www.ceptory.com/.well-known/oauth-authorization-server` with dynamic client registration).
   - Opens Ceptory in the user's browser to approve; no secret key to copy. Sessions last 30 days and can be revoked immediately at `/dashboard/mcp`.
2. **Bearer token (headless agents, CI, or stdio bridge)**
   - Create a token at `https://www.ceptory.com/dashboard/mcp`.
   - Pass `Authorization: Bearer <token>` on Streamable HTTP requests, or set `CEPTORY_TOKEN=<token>` when launching `npx -y ceptory-mcp`.

## Client Configuration Snippets

### Claude Code
```bash
# OAuth 2.1 (then run /mcp in Claude Code → Authenticate)
claude mcp add --transport http ceptory https://mcp.ceptory.com/mcp

# Or Bearer token
claude mcp add --transport http ceptory https://mcp.ceptory.com/mcp \
  --header "Authorization: Bearer $CEPTORY_TOKEN"
```

### Claude Desktop & claude.ai
- **Connectors UI (OAuth)**: `Settings → Connectors → Add custom connector` → Name: `Ceptory`, URL: `https://mcp.ceptory.com/mcp`.
- **Token via stdio (`claude_desktop_config.json`)**:
```json
{
  "mcpServers": {
    "ceptory": {
      "command": "npx",
      "args": ["-y", "ceptory-mcp"],
      "env": { "CEPTORY_TOKEN": "<token>" }
    }
  }
}
```

### Cursor (`~/.cursor/mcp.json` or `.cursor/mcp.json`)
```json
{
  "mcpServers": {
    "ceptory": {
      "url": "https://mcp.ceptory.com/mcp"
    }
  }
}
```

### VS Code (`.vscode/mcp.json`)
```json
{
  "servers": {
    "ceptory": {
      "type": "http",
      "url": "https://mcp.ceptory.com/mcp"
    }
  }
}
```

### Codex
```bash
# OAuth
codex mcp add ceptory --url https://mcp.ceptory.com/mcp
codex mcp login ceptory
```
Or with a token:
```toml
[mcp_servers.ceptory]
command = "npx"
args = ["-y", "ceptory-mcp"]
env = { CEPTORY_TOKEN = "<token>" }
```

### Gemini CLI (`~/.gemini/settings.json`)
```json
{
  "mcpServers": {
    "ceptory": {
      "httpUrl": "https://mcp.ceptory.com/mcp"
    }
  }
}
```
Then run `/mcp auth ceptory`.

### Windsurf (`~/.codeium/windsurf/mcp_config.json`)
```json
{
  "mcpServers": {
    "ceptory": {
      "serverUrl": "https://mcp.ceptory.com/mcp"
    }
  }
}
```

## First-Run Verification Workflow

After connecting, run this read-only health check:

1. Call `get_account` to confirm:
   - Signed-in user, workspace name, plan, and role (`OWNER`, `ADMIN`, `MEMBER`, or read-only `VIEWER`).
   - Current prepaid balance and usage before running paid indexing, search, analysis, or AI engine calls.
2. Call `list_cameras` and `list_chats` to discover existing cameras, sites, and video chats.

## MCP Tool Map

| Domain | Tools |
|---|---|
| **Account** | `get_account` |
| **Video chats & search** | `list_chats`, `create_chat`, `get_chat`, `rename_chat`, `delete_chat`, `list_videos`, `get_chat_videos`, `add_video_from_url`, `create_upload_url`, `complete_upload`, `get_video`, `retry_video`, `delete_video`, `search_videos`, `ask_video`, `analyze_video` |
| **Cameras & workflows** | `list_camera_workflows`, `list_cameras`, `get_camera`, `create_camera`, `update_camera`, `delete_camera`, `send_camera_frame`, `get_camera_snapshot`, `test_discord_webhook`, `geocode_address` |
| **People & plates** | `add_camera_person`, `remove_camera_person`, `update_plate_list`, `read_plates` |
| **Sites & movement** | `list_camera_groups`, `create_camera_group`, `update_camera_group`, `delete_camera_group`, `get_movement`, `get_sighting_image` |
| **Alerts** | `list_alerts`, `update_alert`, `get_alert_image` |
| **AI engines** | `list_engines`, `run_engine` |

## Guardrails & Error Handling

- **Confirm destructive actions**: `delete_chat`, `delete_video`, `delete_camera`, `remove_camera_person`, and `delete_camera_group` permanently remove data. Always ask the user before calling them.
- **Role enforcement**: Every MCP tool runs through the same route handler as the dashboard. `VIEWER` roles stay read-only and receive `403` on write tools.
- **Error recovery**:
  - `401 Unauthorized`: MCP token is invalid, expired, or revoked — re-authenticate via OAuth or mint a new token at `/dashboard/mcp`.
  - `402 Payment Required`: Workspace prepaid balance is depleted — top up at `/dashboard/billing`.
  - `429 Too Many Requests`: Rate limit hit — wait 60 seconds and retry.
