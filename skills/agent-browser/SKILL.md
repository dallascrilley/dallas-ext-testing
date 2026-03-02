---
name: agent-browser
description: Browser automation workflow using agent-browser CLI. Navigate, snapshot, interact, and verify web applications.
---

# Agent Browser

## Core Workflow

```
navigate → snapshot (-i) → interact (@refs) → re-snapshot → verify
```

## Command Reference

### Navigation
```bash
agent-browser navigate "<url>"          # Go to URL
agent-browser navigate "<url>" --headed  # Show browser window
```

### Snapshots
```bash
agent-browser snapshot                  # Text snapshot of current page
agent-browser snapshot -i               # With interactive element refs (@e1, @e2, ...)
agent-browser screenshot                # Save screenshot to file
agent-browser screenshot --full         # Full page screenshot
```

### Interactions
```bash
agent-browser interact @e1 click        # Click element
agent-browser interact @e1 type "text"  # Type into element
agent-browser interact @e1 select "opt" # Select dropdown option
agent-browser interact @e1 hover        # Hover over element
agent-browser interact @e1 clear        # Clear input field
```

### Information
```bash
agent-browser get-info @e1              # Get element details
agent-browser get-info @e1 --text       # Get text content
agent-browser get-info @e1 --attrs      # Get attributes
```

### Wait
```bash
agent-browser wait 2000                 # Wait N milliseconds
agent-browser wait "@e1"                # Wait for element to appear
```

### Sessions
```bash
agent-browser session list              # List active sessions
agent-browser session close             # Close current session
```

## Modes

| Mode | Flag | Use Case |
|------|------|----------|
| Headless | (default) | CI, automated verification |
| Headed | `--headed` | Debugging, visual inspection |
| JSON | `--json` | Structured output for parsing |

## Verification Pattern

1. Navigate to the page
2. Snapshot with `-i` to get interactive refs
3. Identify the element to test (e.g., `@e5` is the submit button)
4. Interact with elements in sequence
5. Re-snapshot to capture new state
6. Compare before/after snapshots for expected changes

## Tips

- Always use `-i` flag on snapshots to get interactive refs
- Use `--headed` when debugging unexpected behavior
- Chain interactions: navigate → snapshot → interact → snapshot
- Use `--json` when you need to parse output programmatically
