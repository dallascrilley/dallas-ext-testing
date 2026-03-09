---
name: peekaboo
description: Use when automating or inspecting macOS native UI — clicking elements in desktop apps, typing into windows, capturing screenshots of apps or screens, managing windows/menus, or testing non-browser UI flows.
---

# Peekaboo

macOS UI automation CLI. Capture screens, target UI elements, drive input, manage apps/windows/menus.

**Requires:** Screen Recording + Accessibility permissions. Run `peekaboo permissions` to check.

## Core Flow: See → Click → Type

```bash
# 1. Annotate UI to discover element IDs
peekaboo see --app Safari --annotate --path /tmp/see.png

# 2. Click an annotated element (B3 = button 3 from the annotation)
peekaboo click --on B3 --app Safari

# 3. Type into focused field
peekaboo type "user@example.com" --app Safari

# 4. Tab to next field, type, submit
peekaboo press tab --count 1 --app Safari
peekaboo type "password" --app Safari --return
```

Always run `peekaboo see --annotate` before clicking — identifies targets reliably.

## Common Commands

### Inspection
```bash
peekaboo permissions                         # check required permissions
peekaboo list apps --json                    # list running apps
peekaboo list windows --app "Xcode" --json  # list windows for an app
peekaboo see --annotate --path /tmp/ui.png  # annotated UI map with element IDs
```

### Capture
```bash
peekaboo image --mode screen --screen-index 0 --retina --path /tmp/screen.png
peekaboo image --app Safari --window-title "Dashboard" --analyze "Summarize KPIs"
peekaboo image --mode frontmost --path /tmp/frontmost.png
```

### Interaction
```bash
peekaboo click --on B1 --app "App Name"
peekaboo click --coords 500,300 --app "App Name"
peekaboo type "Hello World" --app "App Name" --delay 10
peekaboo hotkey --keys "cmd,shift,t"
peekaboo press escape
peekaboo scroll --direction down --amount 6 --smooth
peekaboo drag --from B1 --to T2
```

### App & Window Management
```bash
peekaboo app launch "Safari" --open https://example.com
peekaboo window focus --app Safari --window-title "Example"
peekaboo window set-bounds --app Safari --x 50 --y 50 --width 1200 --height 800
peekaboo app quit --app Safari
```

### Menus
```bash
peekaboo menu click --app TextEdit --path "Format > Font > Show Fonts"
peekaboo menu click-extra --title "WiFi"
peekaboo menubar list --json
```

## Targeting Parameters

Most interaction commands accept these targeting options:

- `--app <name>` — target app by name
- `--window-title <title>` — target specific window
- `--window-id <id>` — target by window ID (from `list windows`)
- `--on <id>` / `--id <id>` — element ID from `see --annotate`
- `--coords x,y` — absolute screen coordinates
- `--snapshot <id>` — use a specific snapshot (defaults to latest)

## Output

All commands support `--json` / `-j` for structured output and `--verbose` / `-v` for debug logging.
