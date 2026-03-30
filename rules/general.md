## Environment Variables

When testing code that depends on environment variables:
- Use the test framework's stubbing API (if available) in `beforeEach` and restore in `afterEach`
- Avoid modifying `process.env` (or equivalent) directly — it can leak between tests and is less reliable
- To "unset" a variable, use an empty string

Example (Vitest):

```ts
beforeEach(() => {
  vi.stubEnv('NAME', 'value');
});
afterEach(() => {
  vi.unstubAllEnvs();
});
```

Example (pytest):

```py
def test_something(monkeypatch):
    monkeypatch.setenv('NAME', 'value')
    # ... test runs with NAME set
    # monkeypatch auto-restores in teardown
```
