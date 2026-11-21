# Video MCP Tool Directory & Permissions

All tools execute through the dashboard route handlers with the approving user's workspace, RBAC role (`OWNER`, `ADMIN`, `MEMBER`, or read-only `VIEWER`), rate limits, and prepaid balance.

| Domain | Tool | Mode | Paid / Destructive Notes |
|---|---|---|---|
| **Account** | `get_account` | Read | Returns user, workspace, plan, role, prepaid balance, usage, recent chats. |
| **Video chats** | `list_chats`, `get_chat`, `list_videos`, `get_chat_videos`, `get_video` | Read | Poll `get_chat_videos` until `READY`. `get_video` returns indexed segment timeline. |
| **Video ingest** | `create_chat`, `rename_chat`, `add_video_from_url`, `create_upload_url`, `complete_upload`, `retry_video` | Write | Indexing billed per minute (`retry_video` is free). Stdio `ceptory-mcp` adds `upload_video_file`. |
| **Video query** | `search_videos`, `ask_video`, `analyze_video` | Write | Uses workspace prepaid balance (`402` if depleted). |
| **Video delete** | `delete_chat`, `delete_video` | **Destructive** | Ask user before calling. |
| **Cameras** | `list_camera_workflows`, `list_cameras`, `get_camera`, `get_camera_snapshot`, `geocode_address` | Read | Snapshot returns latest frame image + 10-min signed link. |
| **Camera config** | `create_camera`, `update_camera`, `send_camera_frame`, `test_discord_webhook` | Write | `update_camera` replaces `config` — always merge from `get_camera` first. |
| **Camera delete** | `delete_camera` | **Destructive** | Removes camera, alerts, and enrolled people. Ask user first. |
| **People & plates** | `add_camera_person`, `update_plate_list`, `read_plates` | Write / Read | `read_plates` is read-only. Up to 5 photos/person; up to 10 named plate lists. |
| **People delete** | `remove_camera_person` | **Destructive** | Ask user before calling. |
| **Sites & Re-ID** | `list_camera_groups`, `get_movement`, `get_sighting_image` | Read | Requires `track_people: true` on cameras for movement journeys. |
| **Site config** | `create_camera_group`, `update_camera_group` | Write | Assign cameras using `update_camera({ camera_id, group_id })`. |
| **Site delete** | `delete_camera_group` | **Destructive** | Cameras remain ungrouped. Ask user first. |
| **Alerts** | `list_alerts`, `get_alert_image`, `update_alert` | Read / Write | Inspect `get_alert_image` before resolving false alarms. |
| **AI engines** | `list_engines`, `run_engine` | Read / Write | `run_engine` billed against prepaid balance; supports `annotate: true`. |
