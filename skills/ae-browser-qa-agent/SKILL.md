---
name: ae-browser-qa-agent
description: "Parallel browser QA testing using Playwright CLI with natural language user stories. Use when the user wants to test UI flows, validate user stories against a web app, run browser-based acceptance tests, or set up agentic QA automation. Also use for \\\\\\\"browser testing\\\\\\\", \\\\\\\"UI review\\\\\\\", \\\\\\\"QA automation\\\\\\\", \\\\\\\"user story validation\\\\\\\", \\\\\\\"agentic testing\\\\\\\", or when someone says \\\\\\\"test my app\\\\\\\" or \\\\\\\"check if the UI works\\\\\\\"."
---

# Browser QA Agent

Orchestrate parallel browser testing where each user story runs in its own sub-agent,
takes screenshots at every step, and reports structured pass/fail results back to
an orchestrator.

This skill teaches a 4-layer architecture for scalable, repeatable browser QA:

1. **Skill** — raw browser capability (Playwright CLI)
2. **Agent** — specialized QA workflow per story
3. **Command** — orchestration prompt that fans out agents
4. **Justfile** — reusable entry points for humans and CI

## When to use this

- Testing UI flows against localhost, staging, or production
- Validating user stories written in natural language
- Running acceptance tests that behave like real users
- Setting up repeatable QA automation for a project
- Replacing brittle Jest/Vitest browser tests with agentic validation

## Prerequisites

The Playwright CLI (`playwright-cli`) must be installed. This is preferred over
Claude's `--chrome` flag because `playwright-cli` supports parallel sessions —
each sub-agent gets its own isolated browser instance.

If the project already has the `playwright-bowser` or `agent-browser` skill,
this skill builds on top of it. If not, the references below include the
essential CLI commands.

## Architecture Overview

```
justfile (Layer 4: Reusability)
  └── /ui-review command (Layer 3: Orchestration)
        └── bowser-qa-agent × N (Layer 2: Scale)
              └── playwright-cli skill (Layer 1: Capability)
```

The orchestrator discovers user stories from YAML files, spawns one sub-agent
per story, collects results, and produces an aggregate report. Each sub-agent
operates independently — if one story fails, others continue.

## User Story Format

Stories live in YAML files under a configurable directory (default: `ai_review/user_stories/`).

```yaml
stories:
  - name: "Homepage loads with navigation"
    url: "http://localhost:3000"
    workflow: |
      Navigate to http://localhost:3000
      Verify the homepage loads successfully
      Verify a navigation bar is visible with at least 3 links
      Verify a hero section or main content area is present

  - name: "Login flow completes"
    url: "http://localhost:3000/login"
    workflow: |
      Navigate to http://localhost:3000/login
      Verify the login page loads with email and password fields
      Fill in email with "test@example.com"
      Fill in password with "password123"
      Click the login/submit button
      Verify the page redirects to a dashboard or authenticated view
```

Each story has three fields:
- **name** — human-readable label, also used for screenshot directory naming
- **url** — starting URL for the test
- **workflow** — natural language steps the agent walks through sequentially

The workflow format is intentionally flexible. Agents understand imperative steps,
BDD Given/When/Then, checklists, and narrative assertions. Write whatever feels
natural — the agent parses it into discrete steps.

## Setting Up Browser QA in a Project

### Step 1: Create the directory structure

```
your-project/
├── ai_review/
│   └── user_stories/
│       ├── auth.yaml        # Auth-related stories
│       ├── dashboard.yaml   # Dashboard stories
│       └── checkout.yaml    # E-commerce flow stories
├── screenshots/
│   └── bowser-qa/           # Created automatically per run
├── .claude/
│   ├── skills/
│   │   └── playwright-bowser/SKILL.md   # Browser capability
│   ├── agents/
│   │   └── bowser-qa-agent.md           # QA validation agent
│   └── commands/
│       └── ui-review.md                 # Orchestration command
└── justfile                              # Reusable entry points
```

### Step 2: Create the QA agent

The QA agent is a sub-agent definition that handles one user story at a time.
It activates the browser skill, walks through each step, screenshots everything,
and returns a structured report.

See `references/qa-agent-template.md` for the full agent template.

Key design decisions in the agent:
- **Sequential execution** — steps run in order; failure stops remaining steps (marked SKIPPED)
- **Screenshot trail** — every step produces a numbered screenshot for debugging
- **Structured report** — consistent format so the orchestrator can parse results programmatically
- **Console capture on failure** — JS console errors are grabbed when a step fails

### Step 3: Create the orchestration command

The orchestration command (`/ui-review`) is a prompt that:

1. Discovers all YAML story files via glob
2. Generates a timestamped run directory for screenshots
3. Spawns one sub-agent per story (all in parallel via teams)
4. Collects results as agents complete
5. Produces an aggregate pass/fail report

See `references/orchestrator-template.md` for the full command template.

The orchestrator uses **metaprompt engineering** — it teaches each sub-agent exactly
how to format its response so results can be reliably parsed and aggregated.

### Step 4: Add justfile recipes

```just
# UI Review — parallel user story validation
ui-review headed="headed" filter="" *flags="":
    claude --dangerously-skip-permissions --model opus \
      "/ui-review {{headed}} {{filter}} {{flags}}"

# Single QA story validation
test-qa headed="true" prompt="":
    claude --dangerously-skip-permissions --model opus \
      "Use a @bowser-qa-agent: (headed: {{headed}}) {{prompt}}"
```

## The HOP (Higher-Order Prompt) Pattern

For browser automation tasks (not just QA), the HOP pattern wraps variable
workflows in consistent execution logic — like a function that takes a function
as a parameter.

```
/hop-automate <workflow-name> [prompt] [options]
```

The HOP prompt:
1. Loads the named workflow file from a commands directory
2. Applies consistent setup (session naming, screenshot paths, skill selection)
3. Executes the workflow with any variable substitutions
4. Reports results in a standard format

This separates **what to do** (the workflow file) from **how to do it**
(the HOP wrapper). You can add new automation workflows — Amazon purchasing,
blog scraping, data entry — without rewriting the execution logic.

Example workflow file (`amazon-add-to-cart.md`):
```markdown
1. Navigate to https://www.amazon.com
2. Search for: {PROMPT}
3. Click into the first relevant result
4. Click "Add to Cart"
5. Proceed to checkout
6. STOP — do not submit the order
```

## Token and Performance Considerations

- **Playwright CLI over MCP server** — CLI-based interaction is significantly more
  token-efficient because it doesn't load tool schemas into context
- **Parallel sessions** — each sub-agent gets its own named session (`-s=<name>`)
  so stories run concurrently without interference
- **Claude `--chrome` limitation** — Chrome tools cannot run in parallel; only use
  for single-session automations like logged-in purchasing workflows
- **Expect high token usage** — browser QA is inherently expensive (a full Amazon
  workflow took ~80K tokens over 20 minutes on Opus). Budget accordingly.
- **Headless by default** — reduces overhead; use `--headed` only for debugging

## Sub-Agent Report Format

Each QA sub-agent returns results in this structure so the orchestrator can parse them:

```
RESULT: {PASS|FAIL} | Steps: {passed}/{total}

**Story:** {story name}
**Steps:** {passed}/{total} passed

| # | Step             | Status  | Screenshot       |
|---|------------------|---------|------------------|
| 1 | Step description | PASS    | 00_step-name.png |
| 2 | Step description | FAIL    | 01_step-name.png |
| 3 | Step description | SKIPPED | —                |
```

The `RESULT:` line is the machine-parseable summary. The table provides
human-readable detail. On failure, the agent includes a failure detail section
with expected vs actual behavior and any JS console errors.

## Reference Files

| File | When to read |
|------|-------------|
| `references/qa-agent-template.md` | Creating the QA sub-agent definition |
| `references/orchestrator-template.md` | Creating the orchestration command |
| `references/story-examples.md` | Writing user stories in various formats |
