# Orchestrator Template

Copy this into `.claude/commands/ui-review.md` and adapt to your project.

```markdown
---
model: opus
description: Parallel user story validation — discovers YAML stories, fans out QA agents, aggregates results
argument-hint: [headed] [filename-filter] [vision]
---

# UI Review

Discover user stories from YAML files, fan out parallel QA agent instances to validate
each story, then aggregate and report pass/fail results with screenshots.

## Variables

HEADED: $1 (default: "false" — set to "true" or "headed" for visible browser windows)
VISION: detected from $ARGUMENTS — if "vision" appears, enable vision mode. Default: false.
FILENAME_FILTER: remaining non-keyword arguments after removing "vision"
STORIES_DIR: "ai_review/user_stories"
STORIES_GLOB: "ai_review/user_stories/*.yaml"
AGENT_TIMEOUT: 300000
SCREENSHOTS_BASE: "screenshots/bowser-qa"
RUN_DIR: "{SCREENSHOTS_BASE}/{YYYYMMDD_HHMMSS}_{short-uuid}" (generated at start)

## Workflow

### Phase 1: Discover

1. Glob for all files matching STORIES_GLOB
2. If FILENAME_FILTER provided, filter to files whose name contains that substring
3. Read each YAML file and parse the `stories` array
4. Skip unparseable files with a warning
5. Build a flat list of all stories, tracking source file
6. If no stories found, report and stop
7. Generate RUN_DIR:
   ```bash
   RUN_DIR="screenshots/bowser-qa/$(date +%Y%m%d_%H%M%S)_$(uuidgen | tr '[:upper:]' '[:lower:]' | head -c 6)"
   ```

### Phase 2: Spawn

8. Create an agent team
9. For each story, spawn a QA agent with this prompt:

   Execute this user story and report results:

   **Story:** {story.name}
   **URL:** {story.url}
   **Headed:** {HEADED}
   **Vision:** {VISION}

   **Workflow:**
   {story.workflow}

   Instructions:
   - Follow each step sequentially
   - Screenshot after each significant step
   - Save ALL screenshots to: {SCREENSHOT_PATH}
   - Report each step as PASS or FAIL
   - Use this exact format for your final summary:
     RESULT: {PASS|FAIL} | Steps: {passed}/{total}

10. Launch ALL agents in a single message for parallel execution

### Phase 3: Collect

11. Wait for agent completions
12. Parse each report for RESULT line and step counts
13. Mark tasks completed

### Phase 4: Report

14. Clean up the agent team
15. Produce aggregate report:

# UI Review Summary

**Run:** {date and time}
**Stories:** {total} total | {passed} passed | {failed} failed
**Status:** ✅ ALL PASSED | ❌ PARTIAL FAILURE | ❌ ALL FAILED

## Results

| #   | Story        | Source File | Status  | Steps            |
| --- | ------------ | ----------- | ------- | ---------------- |
| 1   | {story name} | {filename}  | ✅ PASS  | {passed}/{total} |
| 2   | {story name} | {filename}  | ❌ FAIL  | {passed}/{total} |

## Failures
(Only if failures exist — include full agent report for each)

## Screenshots
All screenshots saved to: `{RUN_DIR}/`
```
