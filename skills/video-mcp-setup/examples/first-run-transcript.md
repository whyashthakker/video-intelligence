# Example: First-Run MCP Verification & Error Recovery

## Scenario
A developer just added `https://mcp.ceptory.com/mcp` to Claude Code and asks: *"Check if the video MCP server is working and show me what's in my workspace."*

## Step 1 — Verify Account & Balance
Tool call: `get_account({})`

Expected response summary to report back:
- **Workspace**: `Acme Logistics` (`DEVELOPER` plan, role: `OWNER`)
- **Balance / Allowance**: `$48.20` prepaid balance + remaining free indexing minutes
- **Recent activity**: 2 video chats

## Step 2 — Check Cameras & Video Chats
Tool calls (in parallel):
- `list_cameras({})`
- `list_chats({})`

Report concisely:
- 3 cameras connected (`Loading Dock` online, `North Gate` online, `East Yard` offline — 2 open alerts).
- 2 video chats (`Night Shift Exports` READY, `Drone Flight Nov 20` PROCESSING).

## Error Recovery Patterns
- **If `get_account` returns `401`**: Tell the user their MCP session expired or token was revoked; instruct them to run `/mcp` → Authenticate (OAuth) or create a new token at `https://www.ceptory.com/dashboard/mcp`.
- **If a paid tool returns `402`**: Stop batch operations and direct the user to `https://www.ceptory.com/dashboard/billing` to add balance.
- **If a write tool returns `403`**: Confirm the user's `role` from `get_account` — `VIEWER` accounts are read-only.
- **If any tool returns `429`**: Wait 60 seconds before retrying and reduce parallel tool concurrency.
