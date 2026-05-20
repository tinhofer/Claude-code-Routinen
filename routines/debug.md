---
name: debug
purpose: Systematically reproduce, isolate, and fix a bug.
inputs:
  - Bug description or failing test
  - Steps to reproduce (if known)
when-to-use: When the failure mode is unclear or non-deterministic.
---

# Debug

Investigate `{{bug}}`. Resist the urge to guess — confirm each
hypothesis with evidence before acting on it.

## Steps

1. **Reproduce.** Find or create the smallest input that triggers the
   bug. If you can't reproduce, say so and stop.
2. **Localize.** Bisect: which commit, which file, which line. Use
   `git log -S`, `git bisect`, or targeted reads.
3. **Hypothesize.** State the suspected root cause in one sentence,
   plus what evidence would confirm or refute it.
4. **Verify.** Add a logging statement, a failing test, or a minimal
   experiment that distinguishes the hypothesis from alternatives.
5. **Fix.** Apply the minimum change that resolves the root cause —
   not the symptom. Avoid unrelated cleanup.
6. **Regress.** Add a test that fails without the fix and passes with
   it.

## Output format

```
Repro: <command or steps>
Localized to: <file:line>
Root cause: <one sentence>
Evidence: <how you confirmed it>
Fix: <description + diff summary>
Regression test: <test name>
```

## Guardrails

- Do not silence the symptom (try/except, default values) without
  understanding the root cause first.
- Do not refactor surrounding code in the same change.
- If the fix touches >3 files, pause and explain why.
