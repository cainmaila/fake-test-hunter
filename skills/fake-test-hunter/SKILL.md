---
name: fake-test-hunter
description: Audit whether tests are meaningful and list fake tests (read-only). Use for any review of test quality or effectiveness.
context: fork
agent: general-purpose
allowed-tools: Read, Grep, Glob, Bash
---

# Fake Test Hunter

Scope: if an argument is given, audit that path; otherwise audit every test the project's test command runs.

List the full set of test files with Glob first, then review every file. No sampling.

## Criteria

**Valid** (any one):

1. **Deep**: the input goes through meaningful logic before reaching the output, and the assertion checks the result of that logic.
2. **Broad**: covers extreme values and exceptions, not just the happy path. A `describe` block that is broad as a whole counts.

**Fake** (any one):

1. **Obvious**: the answer is known without reading the code. Easy-to-miss forms: asserting whatever the mock returns; only checking "exists / not empty / was called" (e.g. `toBeDefined`, `assertIsNotNone`) without checking the result; asserting something the type system or compiler already guarantees.
2. **Neither deep nor broad**: meets neither validity criterion.

Judging depth requires reading the source under test (follow imports). Do not judge from the test alone.

## Output

List fake tests only, grouped by file (the template below is format only, not items to look for):

```
### <file path>
| Line | Test name | Type | Reason |
|------|-----------|------|--------|
| <line> | '<test name>' | Obvious / Neither deep nor broad | <one-sentence reason> |
```

End with: `Reviewed N files / M tests, K fake tests`. If none are found, say so. Do not pad the list.
