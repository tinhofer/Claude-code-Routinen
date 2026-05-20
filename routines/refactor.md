---
name: refactor
purpose: Plan and execute a refactor with reversibility checks.
inputs:
  - Target symbol, file, or module
  - Desired end state (e.g. "extract to its own module", "rename")
when-to-use: When the change is mechanical but spans multiple files.
---

# Refactor

Refactor `{{target}}` to `{{desired-state}}`. The behavior must not
change — tests are the source of truth.

## Steps

1. Locate every reference to the target (definition + call sites).
   Report the count before changing anything.
2. Confirm there is at least one test exercising the target. If not,
   stop and ask whether to add one first.
3. Propose the refactor as a sequence of small, independently
   reviewable steps. Wait for confirmation if any step is non-trivial.
4. Apply the steps. After each step, run the test suite (or the
   smallest relevant subset) and report pass/fail.
5. Run the type checker and linter at the end.

## Output format

For the plan:
```
References found: <count> across <n> files
Steps:
  1. <step> — touches <files>
  2. ...
Risks: <bullets>
```

For execution: a brief status after each step, then a final diff
summary.

## Guardrails

- Do not change behavior. If you find a bug, surface it separately
  rather than silently fixing it.
- Do not commit or push without being asked.
- If references span code you cannot see (e.g. another repo), stop and
  flag it.
