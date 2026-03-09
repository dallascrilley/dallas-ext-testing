---
name: pinchtab
description: Use when automating a browser via Pinchtab's local HTTP API — navigating pages, filling forms, clicking elements, scraping content, or taking screenshots using stable accessibility refs.
---

# Pinchtab

Browser control for agents via HTTP + accessibility tree. Runs entirely locally on port 9867.

**Security:** Use a dedicated empty Chrome profile. Never point at your daily profile. Set `BRIDGE_TOKEN` when exposing outside localhost.

## Setup

```bash
# Recommended secure start
BRIDGE_BIND=127.0.0.1 \
BRIDGE_TOKEN="your-strong-secret" \
BRIDGE_PROFILE=~/.pinchtab/automation-profile \
pinchtab &

# Headed (visible window for debugging)
BRIDGE_HEADLESS=false pinchtab &
```

## Core Agent Loop

```
navigate → snapshot → act on refs → snapshot → repeat
```

Refs (`e0`, `e5`, `e12`) are stable after each snapshot — no need to re-snapshot before every action. Only re-snapshot after significant page changes.

**Wait 3+ seconds after navigate before snapshotting** — Chrome needs time to render the accessibility tree.

```bash
curl -X POST http://localhost:9867/navigate \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com"}' && \
sleep 3 && \
curl "http://localhost:9867/snapshot?filter=interactive&format=compact"
```

## CLI Commands

```bash
pinchtab nav https://example.com
pinchtab snap -i -c                    # interactive + compact
pinchtab click e5
pinchtab type e12 "search text"
pinchtab press Enter
pinchtab text                          # readable page text (~800 tokens)
pinchtab ss -o page.jpg                # screenshot
pinchtab eval "document.title"         # run JavaScript
pinchtab pdf --tab TAB_ID -o page.pdf
```

## Snapshot Format

```json
{
  "refs": [
    {"id": "e0", "role": "link", "text": "Sign In"},
    {"id": "e1", "role": "textbox", "label": "Email"},
    {"id": "e2", "role": "button", "text": "Submit"}
  ],
  "text": "... readable page text ...",
  "title": "Login Page"
}
```

## Token Cost Guide

| Method | Approx tokens | When to use |
|--------|--------------|-------------|
| `/text` | ~800 | Reading page content |
| `/snapshot?filter=interactive&format=compact` | ~3,600 | Finding buttons/links |
| `/snapshot?diff=true` | varies | Multi-step (only changes) |
| `/snapshot` | ~10,500 | Full page understanding |
| `/screenshot` | ~2K (vision) | Visual verification |

**Default strategy:** Start with `?filter=interactive&format=compact`. Use `?diff=true` on follow-up snapshots. Use `/text` when only reading content.

## Tips

- Always pass `tabId` explicitly when working with multiple tabs
- Refs persist between snapshot and action — no need to re-snapshot before clicking
- Use `BRIDGE_BLOCK_IMAGES=true` or `"blockImages": true` on navigate for read-heavy tasks
- `BRIDGE_NO_RESTORE=true` disables tab persistence across restarts
