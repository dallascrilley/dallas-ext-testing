---
description: Run browser tests on changed files using agent-browser
allowed-tools: Bash, Read, Glob, Grep
---

# Browser Test

Test browser behavior for: $ARGUMENTS

## Instructions

1. **Identify what to test**:
   - If argument is a URL or route: test that directly
   - If no argument: find changed files with `git diff --name-only HEAD~1`
   - Map changed files to likely routes/pages (check for route definitions, page components)

2. **Choose mode**:
   - Ask user: headed (for debugging, shows browser) or headless (for CI, no UI)?
   - Default: headless

3. **Run agent-browser**:
   ```bash
   # Headless (default)
   agent-browser navigate "<url>"

   # Headed (debugging)
   agent-browser --headed navigate "<url>"
   ```

4. **Snapshot-based verification**:
   ```bash
   # Take initial snapshot with interactive refs
   agent-browser snapshot -i

   # Interact with elements
   agent-browser interact @e1 click
   agent-browser interact @e2 type "test input"

   # Re-snapshot to verify changes
   agent-browser snapshot -i
   ```

5. **Report results**:
   - Pages visited and their visual state
   - Interactions performed and outcomes
   - Any errors or unexpected behavior
   - Screenshots saved for reference
