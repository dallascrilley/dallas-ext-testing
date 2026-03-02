# Browser Testing

## Tool Preference

Prefer `agent-browser` CLI over Playwright MCP server for E2E browser testing. Agent-browser provides a simpler snapshot-based verification workflow that integrates better with Claude Code's agent model.

## Verification Pattern

Use snapshot-based verification:
1. Take a snapshot before the action
2. Perform the action
3. Take a snapshot after the action
4. Compare snapshots for expected changes

## Modes

- **Headed mode** (`--headed`): Use for debugging failing tests or exploring new pages
- **Headless mode** (default): Use for CI pipelines and automated verification
- **JSON mode** (`--json`): Use when parsing output in scripts or agents

## Constraints

- Always snapshot with `-i` flag to get interactive element references
- Don't hardcode element selectors — use snapshot refs (`@e1`, `@e2`)
- Close sessions when done to free resources
