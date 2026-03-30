# Alert Hook — Setup Guide

A Claude Code hook that plays notification sounds and TTS messages for Stop and PermissionRequest events.

## Prerequisites

- Python 3.8+
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)

## 1. Install dependencies

```bash
pip install edge-tts
```

## 2. Copy files

Copy the `claudecodealert/` folder into your project's `.claude/hooks/` directory:

```
your-project/
  .claude/
    hooks/
      claudecodealert/
        scripts/
          alert.py
          sound/
            bell.mp3
            noti.mp3
        README.md
        SETUP.md
```

## 3. Add hooks to settings.json

Open your project's `.claude/settings.json` (create it if it doesn't exist) and add both hooks:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "REPO=$(git rev-parse --show-toplevel) && python \"$REPO/.claude/hooks/claudecodealert/scripts/alert.py\"",
            "timeout": 30
          }
        ]
      }
    ],
    "PermissionRequest": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "REPO=$(git rev-parse --show-toplevel) && python \"$REPO/.claude/hooks/claudecodealert/scripts/alert.py\" --message \"Permission needed.\" --sound noti",
            "timeout": 15
          }
        ]
      }
    ]
  }
}
```

> If `python` is not on your PATH, replace it with the full path to your Python binary (e.g., `python3`, `/usr/bin/python3`, `C:/Users/you/miniconda3/python.exe`).

## 4. Add the TTS tag rule to CLAUDE.md

Add the following to your project's `CLAUDE.md` so Claude includes a spoken summary in every response:

```markdown
### Stop Alert — TTS tag requirement

**Every response MUST end with a `<!-- tts: ... -->` tag** on its own line. The Stop hook parses this tag to generate a spoken notification. If the tag is missing, the user hears only a generic "Task completed" bell.

Format:
\```
<!-- tts: {"m": "Done. Short summary of what was done."} -->
\```

Rules:
- `m` value must be a single English sentence, max 15 words.
- Start with "Done." followed by a brief summary.
- NEVER include file paths, URLs, code snippets, or slashes.
- NEVER include Korean — always English.
- Be conversational and natural — this is read aloud.
- Even for simple Q&A or conversational replies, include the tag.
```

## 5. Verify

Start a new Claude Code session and send any message. You should hear:
- **On response**: bell chime + spoken summary
- **On permission request**: notification chime + "Permission needed."

If you only hear the bell (no TTS), check `.claude/hooks/claudecodealert/scripts/alert_debug.json` for errors.

## Troubleshooting

| Symptom | Fix |
|---|---|
| No sound at all | Verify `python` runs and `edge-tts` is installed: `python -c "import edge_tts"` |
| Bell plays but no TTS | Check `alert_debug.json` for `TTS_ERROR`. Usually a network issue (edge-tts needs internet). |
| Generic "Task completed" instead of summary | Claude didn't include the `<!-- tts: ... -->` tag. Verify your CLAUDE.md has the TTS tag rule. |
| Hook timeout | Increase `"timeout"` in settings.json (default 30s). |
