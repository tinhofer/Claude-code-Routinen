---
name: code-review
purpose: Produce a structured review of a diff or pull request.
inputs:
  - A diff, PR number, or branch name
when-to-use: Before merging, or when asked for a second opinion on a change.
---

# Code Review

Review the change at `{{target}}` (PR number, branch, or diff). Read every
file in the diff before commenting — do not skim.

## Steps

1. Summarize the change in 1–2 sentences: what it does and why.
2. Identify the **risk surface**: which systems, callers, or invariants
   this change touches.
3. Walk the diff file by file. For each finding, note severity:
   - **blocker** — incorrect, unsafe, or breaks contracts
   - **important** — likely bug, missing edge case, or significant
     design concern
   - **nit** — style, naming, or minor cleanup
4. Check explicitly: error handling at boundaries, concurrency, input
   validation, tests covering the new behavior, backwards compatibility.
5. End with a verdict: **approve**, **approve with changes**, or
   **request changes**.

## Output format

```
## Summary
<1–2 sentences>

## Risk surface
- <bullet>

## Findings
### <file:line> — <severity>
<what's wrong and the suggested fix>

## Verdict
<approve | approve with changes | request changes>
```

## Guardrails

- Do not push commits or post to GitHub. Output the review only.
- If the diff is huge (>500 lines), ask whether to focus on a subset.
- Flag anything that looks like a secret or credential immediately.
