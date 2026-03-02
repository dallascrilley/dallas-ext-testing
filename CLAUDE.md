# dallas-ext-testing

> L1 extension plugin providing browser-based and E2E testing tools. Extends L0 testing discipline from dallas-base-plugin.

## Structure

- `commands/` — test-browser command for agent-browser workflow
- `skills/` — agent-browser skill for snapshot-based verification
- `rules/` — browser testing conventions
- `agents/browser-tester.md` — Browser testing agent for mock IPC verification with agent-browser

## Prerequisites

- `agent-browser` CLI: `npm install -g agent-browser && agent-browser install`
