---
name: test-squad
description: Dispatch test specialists to plan and write deep, broad tests in parallel, then verify in the main thread. Use whenever tests need to be added, strengthened, or fixed, including fake-test-hunter reports.
---

# Test Squad

Produce real tests: specialists write in parallel, the main thread verifies personally. If no scope is given, ask first; never scan everything. Can take a `fake-test-hunter` report as input.

## Criteria

- Valid (any one): **deep** = the assertion checks the result of the logic under test; **broad** = covers extreme values and error paths.
- Fake: **obvious** (answer is known without reading the code), or **neither deep nor broad**.

## Principles

1. A test is worth something only if it FAILS when the source is broken. A valid mutation: changes the logic the test claims to verify, still loads after the change, the FAIL comes from the assertion, and it returns to PASS after reverting.
2. Sources of false passes: expected value equals the default; the assertion actually comes from a mock return; a one-shot mock is consumed by initialization before the code under test calls it. Pick data so none of these paths can pass on their own.
3. A new test that FAILS against the current code: do not edit the assertion to fit. Report it as a suspected bug and let the user decide.
4. A specialist's self-report is only a lead. Conclusions rest on the main thread's verification.

## Process (no step may be skipped)

1. **Baseline**: if the source under test has uncommitted changes, stop and ask. Record the working tree state, plus the failure set of the full test suite and static checks (if any).
2. **Plan**: decide an action per item; dispatch only after the user confirms.
   - Delete: obvious, with no assertion worth keeping.
   - Strengthen: neither deep nor broad, or violates Principle 2.
   - Add: branches, extreme values, or error paths not covered.
3. **Dispatch**: parallel subagents, each in an isolated workspace (e.g. a worktree) so mutations do not contaminate each other; each test file belongs to one specialist only. The prompt includes the criteria, the full principles, and the report format. Specialists edit test files only; the sole exception is temporarily mutating source, which must be reverted before reporting. Every added or strengthened test gets a mutation, reported as "which logic was changed → assertion failure message".
4. **Verify** (main thread):
   - After collecting the test files, the working tree differs from the baseline only by test file changes.
   - The failure set of the full suite and static checks ⊆ the baseline failure set.
   - Redo the mutation for ≥1 test per specialist and ≥3 in total (all of them if fewer); if any mismatch, redo all of that specialist's work.

## Output

Action counts and change in test count, a per-file action table, verification results, suspected bugs, and known limitations (e.g. code mutation cannot reach).
