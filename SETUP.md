# Alert Hook — Setup Guide

A Claude Code hook that plays notification sounds and TTS messages for Stop and PermissionRequest events.

## Prerequisites

- Python 3.8+
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)

## 1. Install dependencies

```bash
pip install edge-tts
```

## 2. Copy `.claude/` into your project

This repo is structured so the `.claude/` folder can be merged directly into your project root:

```
your-project/
  .claude/
    hooks/
      alert/
        scripts/
          alert.py
        sounds/
          bell.mp3
          noti.mp3
    settings.local.json     # Hook registration included
```

The `settings.local.json` already contains the hook commands and permissions — no manual JSON editing needed.

> If `python` is not on your PATH, edit the `command` fields in `.claude/settings.local.json` to use the full path (e.g., `python3`, `/usr/bin/python3`).

## 3. Add the TTS tag rule to CLAUDE.md

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

## 4. Verify

Start a new Claude Code session and send any message. You should hear:
- **On response**: bell chime + spoken summary
- **On permission request**: notification chime + "Permission needed."

If you only hear the bell (no TTS), check `.claude/hooks/alert/scripts/alert_debug.json` for errors.

## Troubleshooting

| Symptom | Fix |
|---|---|
| No sound at all | Verify `python` runs and `edge-tts` is installed: `python -c "import edge_tts"` |
| Bell plays but no TTS | Check `.claude/hooks/alert/scripts/alert_debug.json` for `TTS_ERROR`. Usually a network issue. |
| Generic "Task completed" instead of summary | Claude didn't include the `<!-- tts: ... -->` tag. Verify your CLAUDE.md has the TTS tag rule. |
| Hook timeout | Increase `"timeout"` in `.claude/settings.local.json` (default 30s). |
