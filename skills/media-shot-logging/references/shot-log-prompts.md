# Shot Log Prompts & Channel Cut-Length Presets

## Standard Assistant-Editor Prompt (`analyze_video`)
```text
You are an assistant editor logging this recording. Write:

1. Shot log: a table with columns Timecode in, Timecode out, Title (five words or fewer), Description, and Suggested use (social clip, trailer, recap, archive only).
2. Top five moments, ranked, with one line on why each works.
3. Quotes worth pulling: verbatim lines with timecodes.
4. A suggested order for a 60-second highlight cut, listing the timecodes to use.
5. Tags for the archive: topics, settings, and the roles of people on screen (host, speaker, player), without names unless they are spoken or shown on screen.

Keep timecodes as HH:MM:SS and keep every clip between 5 and 30 seconds.
```

## Channel Cut-Length Presets
Adjust the final line of the prompt to match distribution format:
- **Reels / Shorts / TikTok**: *"Keep every clip between 6 and 15 seconds, opening on a visual hook or spoken punchline."*
- **Event recap / YouTube highlights**: *"Keep clips between 20 and 30 seconds so each moment has setup and reaction."*
- **Newsroom / press b-roll**: *"Keep clips 8–12 seconds of clean visual action without cutaways, plus separate verbatim soundbites."*

## Consistent Archive Tagging Taxonomy
Reuse a flat, predictable set of tags across uploads in the same index:
- **Setting**: `stage`, `panel`, `booth-demo`, `crowd`, `sideline`, `backstage`, `screen-share`.
- **Roles** (only use real names when spoken or on a lower-third): `host`, `keynote-speaker`, `panelist`, `interviewer`, `attendee`, `player`.
- **Use**: `social-clip`, `trailer`, `recap`, `b-roll`, `archive-only`.
