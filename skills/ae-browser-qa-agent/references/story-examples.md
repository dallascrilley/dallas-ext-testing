# User Story Examples

## Basic YAML structure

Every story file follows the same schema:

```yaml
stories:
  - name: "Descriptive name for the test"
    url: "https://your-app.com/page"
    workflow: |
      Step-by-step instructions in natural language
```

## Format variations

The QA agent understands multiple story formats. Use whichever feels natural.

### Imperative steps (most common)

```yaml
stories:
  - name: "Front page loads with posts"
    url: "https://news.ycombinator.com/"
    workflow: |
      Navigate to https://news.ycombinator.com/
      Verify the front page loads successfully
      Verify at least 10 posts are visible, each with a title and a link

  - name: "Navigate to page two and back"
    url: "https://news.ycombinator.com/"
    workflow: |
      Navigate to https://news.ycombinator.com/
      Verify the front page loads with posts
      Click the 'More' link at the bottom of the page
      Verify page 2 loads with a new set of posts
      Click the browser back button
      Verify page 1 loads again with the original posts
```

### BDD Given/When/Then

```yaml
stories:
  - name: "Dashboard shows widgets"
    url: "http://localhost:3000"
    workflow: |
      Given I am logged into http://localhost:3000
      When I navigate to /dashboard
      Then I should see a list of widgets with columns: name, status, value
      And each widget should have a numeric value
```

### Narrative with assertions

```yaml
stories:
  - name: "Widget values are valid"
    url: "http://localhost:3000/dashboard"
    workflow: |
      As a logged-in user on http://localhost:3000, go to the dashboard.
      Assert: the page title contains "Dashboard".
      Assert: at least 3 widgets are visible.
      Assert: the top widget has a value under 100.
```

### Checklist with auth

```yaml
stories:
  - name: "Dashboard validation"
    url: "http://localhost:3000/dashboard"
    workflow: |
      url: http://localhost:3000/dashboard
      auth: user@test.com / secret123
      - [ ] Dashboard loads
      - [ ] At least 3 widgets visible
      - [ ] Values are numeric
      - [ ] Clicking a widget opens detail view
```

## Organizing stories by domain

Split stories into files by feature area:

```
ai_review/user_stories/
├── auth.yaml           # Login, logout, password reset
├── dashboard.yaml      # Dashboard widgets and charts
├── checkout.yaml       # E-commerce purchase flow
├── settings.yaml       # User settings and preferences
└── onboarding.yaml     # New user onboarding wizard
```

The orchestrator discovers all `.yaml` files automatically. Use the filename
filter to run a subset: `/ui-review headed auth` runs only stories from
files matching "auth".

## Tips for writing effective stories

- **Be specific about what "verify" means** — "Verify 10 posts are visible" is
  better than "Verify the page looks right"
- **Include the full URL** in the workflow even if it matches the `url` field —
  the agent uses the workflow steps, not the `url` field, for navigation
- **One flow per story** — a story that logs in, navigates, edits, and logs out
  is harder to debug than four focused stories
- **Name stories descriptively** — names become directory names for screenshots,
  so "Login with valid credentials" beats "Test 1"
