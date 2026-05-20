---
name: daily-standup
purpose: Summarize recent work for a standup or daily log.
inputs:
  - Time window (default: since yesterday 9am)
  - Optional: author filter (default: current git user)
when-to-use: Before standup, or at end of day to write a log entry.
---

# Daily Standup

Summarize what was done in `{{window}}` by `{{author}}`.

## Steps

1. Gather commits: `git log --since="{{window}}" --author="{{author}}"
   --oneline`.
2. Gather open PRs authored or reviewed in the window.
3. Group commits by theme (feature, fix, refactor, infra). Drop noise
   like merge commits and version bumps.
4. Identify **in-flight** work: branches with commits but no merged PR.
5. Identify **blockers**: PRs awaiting review for >24h, failing CI,
   open questions in comments.

## Output format

```
## Yesterday
- <theme>: <one-line summary> (<commit hashes or PR numbers>)

## Today
- <planned item>

## Blockers
- <blocker, with link or PR number>
```

Keep it short — three to five bullets per section is the target.

## Guardrails

- Do not invent work that isn't in the commit history.
- If the window is empty, say so plainly instead of padding.
