# Claude-code-Routinen

A personal library of reusable **routines** (prompts and workflows) for
[Claude Code](https://claude.ai/code). Each routine is a self-contained
Markdown file you can paste into Claude, run as a slash command, or adapt
to your own project.

## Layout

```
routines/
  code-review.md         # Review a diff or PR
  refactor.md            # Guided refactor with safety checks
  debug.md               # Systematic bug hunt
  release-checklist.md   # Pre-release verification
  daily-standup.md       # Summarize yesterday's work
templates/
  routine.md             # Template for new routines
```

## How to use a routine

**Option A — paste into Claude Code**
Open the routine file, copy its body, and paste it into a Claude Code
session. Fill in any `{{placeholders}}`.

**Option B — as a slash command**
Copy the file into your project's `.claude/commands/` directory (or
`~/.claude/commands/` to make it global). The filename becomes the
command name, e.g. `routines/code-review.md` → `/code-review`.

**Option C — reference from CLAUDE.md**
Add a pointer like `See routines/release-checklist.md before tagging` so
the routine is loaded into context when needed.

## Adding a new routine

1. Copy `templates/routine.md` to `routines/<name>.md`.
2. Fill in the front matter (purpose, inputs, when to use).
3. Write the prompt body. Keep it self-contained — the routine should
   work without referring back to this README.
4. Add a row to the index below.

## Index

| Routine | Purpose |
| --- | --- |
| [code-review](routines/code-review.md) | Structured review of a diff or PR |
| [refactor](routines/refactor.md) | Plan and execute a refactor safely |
| [debug](routines/debug.md) | Reproduce, isolate, and fix a bug |
| [release-checklist](routines/release-checklist.md) | Pre-release verification |
| [daily-standup](routines/daily-standup.md) | Summarize recent work for standup |
