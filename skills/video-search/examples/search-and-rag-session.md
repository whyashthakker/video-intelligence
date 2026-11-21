# Example: Multi-Query Footage Search & Synthesis

## Query Quality Comparison

| Weak Query (Avoid) | Strong Query (Use Instead) | Why |
|---|---|---|
| `"suspicious activity"` | `"person climbing over chain-link fence at night"` | Concrete visual action & object. |
| `"John entering office"` | `"person in navy jacket carrying laptop bag entering glass door"` | Describes appearance, not guessed identity. |
| `"safety issue"` | `"worker without a hard hat standing next to moving forklift"` | Names exact missing PPE + hazard context. |
| `"the part about budget"` | `"speaker says Q4 budget is approved"` | Targets indexed speech directly. |

## Worked Investigation Session (MCP)
Given a chat (`chat_id`) containing 3 exported camera clips (`gate.mp4`, `corridor.mp4`, `dock.mp4`):

1. **Decompose into atomic queries**:
   - `search_videos({ chat_id, query: "white sedan pulling up to main gate", min_score: 0.55 })`
   - `search_videos({ chat_id, query: "person in dark hoodie walking past corridor exit", from_sec: 300, to_sec: 1200 })`
   - `search_videos({ chat_id, query: "someone carrying cardboard box out of loading dock" })`
2. **Verify top hits with `ask_video`**:
   - `ask_video({ video_id: "<dock_video_id>", question: "Between 00:08:10 and 00:09:30, how many boxes were moved and which direction did the person exit?" })`
3. **Present grounded output**:
   - `gate.mp4` (`00:05:12–00:05:24`, score `0.88`): White sedan stops outside north barrier.
   - `corridor.mp4` (`00:06:40–00:06:51`, score `0.83`): Person in dark hoodie walks east toward loading dock.
   - `dock.mp4` (`00:08:18–00:08:49`, score `0.91`): Two brown cartons carried through bay 2 toward parking apron. License plate obscured (`unconfirmed`).
