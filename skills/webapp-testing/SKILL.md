---
name: webapp-testing
description: Toolkit for interacting with and testing local web applications using agent-browser. Supports verifying frontend functionality, debugging UI behavior, capturing browser screenshots, and viewing browser state.
---

# Web Application Testing

To test local web applications, use the `agent-browser` CLI.

## Decision Tree: Choosing Your Approach

```
User task → Is it static HTML?
    ├─ Yes → Read HTML file directly to identify structure
    │         ├─ Success → Use agent-browser to navigate file:// URL and verify
    │         └─ Fails/Incomplete → Treat as dynamic (below)
    │
    └─ No (dynamic webapp) → Is the server already running?
        ├─ No → Start the server in the background first,
        │        then use agent-browser to navigate and test
        │
        └─ Yes → Reconnaissance-then-action:
            1. Navigate and wait for page to load
            2. Take snapshot with -i for interactive refs
            3. Identify elements from rendered state
            4. Execute interactions with discovered refs
```

## Core Workflow

```
navigate → snapshot (-i) → interact (@refs) → re-snapshot → verify
```

## Example: Testing a Local Dev Server

**Start server and navigate:**
```bash
# Start dev server in background
npm run dev &

# Navigate to the app
agent-browser navigate "http://localhost:5173"
```

**Inspect the page:**
```bash
# Get interactive snapshot with element refs
agent-browser snapshot -i

# Or take a screenshot for visual inspection
agent-browser screenshot --full
```

**Interact with elements:**
```bash
# Click a button (refs come from snapshot -i output)
agent-browser interact @e3 click

# Fill a form field
agent-browser interact @e1 type "test@example.com"

# Select a dropdown option
agent-browser interact @e5 select "Option B"
```

**Verify results:**
```bash
# Re-snapshot to see updated state
agent-browser snapshot -i

# Get specific element details
agent-browser get-info @e2 --text
```

## Reconnaissance-Then-Action Pattern

1. **Navigate and snapshot**:
   ```bash
   agent-browser navigate "http://localhost:3000"
   agent-browser snapshot -i
   ```

2. **Identify elements** from snapshot output (e.g., `@e1` is the email input, `@e5` is submit)

3. **Execute actions** using discovered refs:
   ```bash
   agent-browser interact @e1 type "user@example.com"
   agent-browser interact @e2 type "password123"
   agent-browser interact @e5 click
   ```

4. **Verify outcome**:
   ```bash
   agent-browser snapshot -i
   # Check for expected changes in the output
   ```

## Common Pitfall

- **Don't** interact with elements before taking a snapshot — refs may not exist yet
- **Do** always `snapshot -i` first to discover available interactive refs

## Best Practices

- Always use `-i` flag on snapshots to get interactive element refs
- Use `--headed` flag when debugging unexpected behavior visually
- Use `--json` flag when you need structured output for programmatic parsing
- Chain the workflow: navigate → snapshot → interact → snapshot → verify
- Use `agent-browser wait "@e1"` to wait for elements that load asynchronously
- Use `agent-browser wait 2000` for time-based waits when element waits aren't sufficient
- Close sessions when done: `agent-browser session close`
