---
name: video-mcp-setup
description: Set up and connect the video intelligence MCP server (hosted at https://mcp.ceptory.com/mcp) or stdio bridge (npx -y ceptory-mcp) in Claude Code, Claude Desktop, claude.ai, Cursor, VS Code, Codex, Gemini CLI, Windsurf, Antigravity, OpenAI Agents SDK, LangChain, Mastra, or n8n. Covers OAuth 2.1 browser sign-in, Bearer tokens, local file upload helpers, workspace RBAC/balance checks, and troubleshooting 401/402/429 errors. Use whenever a user asks to set up the video MCP server, connect an AI agent to video search or cameras, or configure MCP tools for video intelligence and CCTV cameras.
license: MIT
metadata:
  version: "1.0"
---

# Video Intelligence MCP Server Setup

Connect any MCP client or agent framework to the video intelligence MCP server so your agent operates with the approving user's workspace, role, and prepaid balance.

## Endpoints

- **Streamable HTTP (OAuth 2.1 PKCE or Bearer)**: `https://mcp.ceptory.com/mcp`
- **Stdio bridge (adds `upload_video_file` & `image_path`)**: `CEPTORY_TOKEN=<token> npx -y ceptory-mcp`
- **Token management / revoke agents**: `https://www.ceptory.com/dashboard/mcp`
- **Docs**: `https://www.ceptory.com/mcp` · `https://www.ceptory.com/mcp/llms.txt`

> The backend exposes an MCP server and a REST API (`https://www.ceptory.com/api/v1`). Use HTTP directly — do not invent language-specific client SDKs.

## Setup Workflow

1. **Choose transport**:
   - Interactive clients (Claude Code, Claude Desktop, Cursor, VS Code, Codex, Gemini CLI, Windsurf): prefer **OAuth 2.1** via `https://mcp.ceptory.com/mcp` (browser approval, no key to copy).
   - Headless agents, CI, or local file uploads (`upload_video_file` / `image_path`): mint a token at `/dashboard/mcp` and use Bearer auth or `npx -y ceptory-mcp`.
2. **Apply client config**: Read [references/client-configs.md](references/client-configs.md) when generating configuration for a specific client or framework.
3. **Verify connection (read-only)**:
   - Call `get_account` to confirm user, workspace, plan, role (`OWNER`/`ADMIN`/`MEMBER` vs read-only `VIEWER`), and balance.
   - Call `list_cameras` and `list_chats` to inspect available resources.
   - Read [examples/first-run-transcript.md](examples/first-run-transcript.md) for the expected verification flow.
4. **Route tasks to tools**: Read [references/tool-directory.md](references/tool-directory.md) when mapping a request to exact tool names and permission tiers.

## Guardrails & Error Recovery

- **Confirm destructive tools**: Always ask the user before `delete_chat`, `delete_video`, `delete_camera`, `remove_camera_person`, or `delete_camera_group`.
- **`401 Unauthorized`**: Session expired (30-day OAuth) or token revoked — re-authenticate or create a new token at `/dashboard/mcp`.
- **`402 Payment Required`**: Prepaid balance depleted — direct user to `/dashboard/billing`.
- **`403 Forbidden`**: User has read-only `VIEWER` role.
- **`429 Too Many Requests`**: Wait 60 seconds and reduce concurrency.
