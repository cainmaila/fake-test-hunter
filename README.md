<p align="center">
  <img src="assets/banner.png" alt="Fake Test Hunter: a hunter shatters a fake green checkmark with a glowing bolt" width="100%">
</p>

# Fake Test Hunter

**English** | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md)

### Tired of AI lying to you?

You asked your AI to "add tests." It came back with a green checkmark, 40 new tests, and a confident summary.

Then you broke the code on purpose. **Every test still passed.**

That is a fake test: it looks like safety, but it protects nothing.

```ts
// The mock returns 42, then the test asserts... 42.
mockApi.getUser.mockResolvedValue({ id: 42 })
expect((await getUser()).id).toBe(42)

// "It exists." Great. Is it correct?
expect(result).toBeDefined()
```

AI is very good at writing tests that pass. It is much worse at writing tests that **can fail**. This repo gives your agent two skills to close that gap.

## What's inside

| Skill | What it does |
|-------|--------------|
| **`fake-test-hunter`** | Read-only audit. Reads your tests *and* the source they cover, then lists every fake test with file, line, type, and a one-sentence reason. No sampling, no padding. If it finds nothing, it says so. |
| **`test-squad`** | Dispatches specialist subagents in parallel (each in an isolated worktree) to delete, strengthen, or add tests. Then the **main thread verifies the work itself** instead of trusting the specialists' reports. |

## How a test is judged

A test is **valid** if it is either:

- **Deep**: the input passes through real logic, and the assertion checks that logic's result.
- **Broad**: it covers extreme values and error paths, not only the happy path.

A test is **fake** if it is:

- **Obvious**: the answer is known without reading the code (asserting what the mock returned, `toBeDefined`, `assertIsNotNone`, things the type system already guarantees).
- **Neither deep nor broad.**

## Why `test-squad` doesn't just trust the AI

The core rule: **a test is worth something only if it FAILS when the source is broken.**

So every added or strengthened test must come with a mutation: change the logic the test claims to verify, show the assertion failing, revert, show it passing again. Then the main thread:

1. Confirms the working tree differs from the baseline only by test files.
2. Confirms the failure set of the full suite and static checks is a subset of the baseline's.
3. Re-runs mutations itself on a sample from every specialist. One mismatch, and that specialist's work is redone.

If a new test fails against the current code, `test-squad` does **not** bend the assertion to make it pass. It reports a suspected bug and leaves the decision to you.

## Install

```bash
npx skills add cainmaila/fake-test-hunter
```

## Use

```text
/fake-test-hunter                 # audit every test the project runs
/fake-test-hunter src/auth        # audit one path

/test-squad src/auth              # plan, confirm with you, dispatch, verify
```

Typical flow: run the hunter, hand its report to the squad, review the plan, approve, done.

`test-squad` asks for a scope first and never scans everything. It stops if the source under test has uncommitted changes, and it waits for your confirmation before dispatching anything.

## Limits

- The hunter is an LLM review, not a mutation-testing tool. Treat its list as a strong lead, not a verdict.
- Mutation cannot reach every line. `test-squad` reports what it could not cover.
- Verification cost grows with the number of specialists.

## License

[MIT](LICENSE)
