---
name: ceptory-mcp-setup
description: Set up and connect Ceptory's official MCP server (https://mcp.ceptory.com/mcp) or stdio bridge (npx -y ceptory-mcp) in Claude Code, Claude Desktop, claude.ai, Cursor, VS Code, Codex, Gemini CLI, Windsurf, Antigravity, OpenAI Agents SDK, LangChain, Mastra, or n8n. Covers OAuth 2.1 browser sign-in, Bearer tokens, local file upload helpers, workspace RBAC/balance checks, and troubleshooting 401/402/429 errors. Use whenever a user asks to set up Ceptory MCP, connect an AI agent to Ceptory, or configure MCP tools for video intelligence and CCTV cameras.
---

# Ceptory MCP Server Setup & Agent Connection

Connect any Model Context Protocol (MCP) client or agent framework to [Ceptory](https://www.ceptory.com) so your agent can watch and search video, configure live camera workflows, triage alerts, trace cross-camera movement, and run AI detection engines—acting as the signed-in user with their workspace permissions and prepaid balance.

## Official Endpoints & Links

- **MCP Streamable HTTP URL**: `https://mcp.ceptory.com/mcp` (MCP `2026-07-28`; 2025-era clients supported)
- **Stdio bridge (adds `upload_video_file` & `image_path` for local files)**: `CEPTORY_TOKEN=<token> npx -y ceptory-mcp`
- **Interactive setup guide**: [ceptory.com/mcp](https://www.ceptory.com/mcp)
- **Machine-readable MCP docs**: [ceptory.com/mcp/llms.txt](https://www.ceptory.com/mcp/llms.txt)
- **Dashboard tokens & revoke agents**: [ceptory.com/dashboard/mcp](https://www.ceptory.com/dashboard/mcp)
- **Free account signup**: [ceptory.com/signup](https://www.ceptory.com/signup)
- **REST API v1 (no language SDKs; use HTTP directly)**: `https://www.ceptory.com/api/v1`

## Authentication Options

1. **OAuth 2.1 with PKCE (recommended for interactive clients)**
   - Paste only the URL `https://mcp.ceptory.com/mcp`.
   - Clients discover OAuth automatically (`401` → `WWW-Authenticate` `resource_metadata` → `https://www.ceptory.com/.well-known/oauth-authorization-server`, dynamic client registration).
   - Opens Ceptory in the browser so you click **Allow**—no API key to copy. Sessions last 30 days and can be revoked instantly at `/dashboard/mcp`.
2. **Bearer Token (headless agents, CI, servers, or stdio bridge)**
   - Create a token at `https://www.ceptory.com/dashboard/mcp`.
   - Pass `Authorization: Bearer <token>` over Streamable HTTP, or pass `CEPTORY_TOKEN=<token>` to `npx -y ceptory-mcp`.

## Client Setup Recipes

### Claude Code
```bash
# OAuth 2.1 (then run /mcp in Claude Code → Authenticate)
claude mcp add --transport http ceptory https://mcp.ceptory.com/mcp

# Or Bearer token
claude mcp add --transport http ceptory https://mcp.ceptory.com/mcp \
  --header "Authorization: Bearer $CEPTORY_TOKEN"
```

### Claude Desktop & claude.ai
- **Connectors UI (OAuth)**: `Settings → Connectors → Add custom connector` → Name: `Ceptory`, URL: `https://mcp.ceptory.com/mcp`, then click **Connect**.
- **Token via stdio (`claude_desktop_config.json`)** *(also unlocks one-step local file `upload_video_file` and `image_path`)*:
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
*(For Bearer auth, add `"headers": { "Authorization": "Bearer <token>" }`.)*

### VS Code / GitHub Copilot (`.vscode/mcp.json`)
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

### OpenAI Codex CLI
```bash
# OAuth
codex mcp add ceptory --url https://mcp.ceptory.com/mcp
codex mcp login ceptory
```
Or with stdio token in `~/.codex/config.toml`:
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

### Agent Frameworks (OpenAI Agents SDK, LangChain, Mastra, n8n, Antigravity)
- Connect via **Streamable HTTP** to `https://mcp.ceptory.com/mcp` with `Authorization: Bearer <token>`, or spawn **stdio** `npx -y ceptory-mcp` with `CEPTORY_TOKEN=<token>`.

## First-Run Verification Checklist

Immediately after configuring MCP, run these read-only checks:
1. Call `get_account` → confirms signed-in user, workspace, plan, role (`OWNER`/`ADMIN`/`MEMBER` vs read-only `VIEWER`), and prepaid balance / remaining free minutes.
2. Call `list_cameras` → lists connected cameras, online/offline status, workflows, and open alerts.
3. Call `list_chats` → lists existing video chats and indexing statuses.

## Complete MCP Tool Directory

| Domain | Tools |
|---|---|
| **Account** | `get_account` |
| **Video vision, chats & search** | `list_chats`, `create_chat`, `get_chat`, `rename_chat`, `delete_chat`, `list_videos`, `get_chat_videos`, `add_video_from_url`, `create_upload_url`, `complete_upload`, `get_video`, `retry_video`, `delete_video`, `search_videos`, `ask_video`, `analyze_video` *(+ `upload_video_file` in `ceptory-mcp`)* |
| **Cameras & workflows** | `list_camera_workflows`, `list_cameras`, `get_camera`, `create_camera`, `update_camera`, `delete_camera`, `send_camera_frame`, `get_camera_snapshot`, `test_discord_webhook`, `geocode_address` |
| **People & number plates** | `add_camera_person`, `remove_camera_person`, `update_plate_list`, `read_plates` |
| **Sites & cross-camera movement** | `list_camera_groups`, `create_camera_group`, `update_camera_group`, `delete_camera_group`, `get_movement`, `get_sighting_image` |
| **Alerts** | `list_alerts`, `update_alert`, `get_alert_image` |
| **AI vision engines** | `list_engines`, `run_engine` |

## Security, Permissions & Troubleshooting

- **Destructive confirmation**: Always ask the user before calling `delete_chat`, `delete_video`, `delete_camera`, `remove_camera_person`, or `delete_camera_group`.
- **RBAC parity**: MCP executes through the exact dashboard route handlers—`VIEWER` roles stay read-only (`403` on writes).
- **HTTP status troubleshooting**:
  - `401 Unauthorized`: Token invalid, expired (30-day OAuth window), or revoked. Re-run OAuth login or mint a fresh token at [ceptory.com/dashboard/mcp](https://www.ceptory.com/dashboard/mcp).
  - `402 Payment Required`: Workspace prepaid balance exhausted. Top up at [ceptory.com/dashboard/billing](https://www.ceptory.com/dashboard/billing).
  - `429 Too Many Requests`: Rate limit reached. Wait 60 seconds and retry.
