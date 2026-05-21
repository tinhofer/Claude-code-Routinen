# Repo guidelines for Claude

This repo is a personal library of **routines** — self-contained Markdown
prompts for [Claude Code](https://claude.ai/code). Routines live in
`routines/`, their template is in `templates/routine.md`, the index of
all routines is the table at the bottom of `README.md`.

## What "good" looks like here

A routine is a reusable prompt. It must:

1. **Stand alone.** A user should be able to paste the body into a fresh
   Claude session, fill the `{{placeholders}}`, and get a useful result
   without reading any other file in this repo.
2. **Be strict about output format.** The `## Output format` section
   defines the contract. Make it copy-paste-able (code block with the
   exact structure), not prose.
3. **Have guardrails.** Every routine ends with a `## Guardrails`
   section that lists what the routine must *not* do (no pushing, no
   destructive ops without confirmation, no editing prod config, etc.).

## File conventions

- Front matter is required: `name`, `purpose`, `inputs`, `when-to-use`.
  Match the shape in `templates/routine.md` exactly — these fields are
  read by humans skimming the index.
- `name` in the front matter must match the filename (without `.md`).
  `routines/code-review.md` → `name: code-review`.
- Structure of a routine file, in order: YAML front matter
  (delimited by `---`), then the `# <Title>` heading, then `## Steps`,
  `## Output format`, `## Guardrails`. Front matter always comes first;
  don't invent new top-level sections without a reason.
- Wrap prose at ~72 columns. Code blocks and tables don't need wrapping.
- Use plain Markdown. No HTML, no emoji.

## Adding or renaming a routine

When you add `routines/<name>.md`:

1. Copy `templates/routine.md` as the starting point.
2. Add a row to the **Index** table in `README.md` — same order as the
   filename's alphabetical position.
3. If the routine is meant to be used as a slash command, the filename
   becomes the command name. Keep it short and hyphenated
   (`release-checklist`, not `release_checklist` or `ReleaseChecklist`).

When you rename a routine, update both the front matter `name:` field
and the row in the README index.

## Editing existing routines

- Treat the `## Output format` block as a contract. Changing it can
  break downstream usage where someone has already wired the routine
  into a workflow. If you must change it, note the change in the commit
  message (e.g. `code-review: tighten output schema`).
- Keep the Steps numbered and imperative ("Read every file in the diff",
  not "The reviewer should read every file").

## What not to do

- Don't add a routine that doesn't fit the
  *paste-into-Claude-and-go* model — this repo is a prompt library, not
  a code library.
- Don't introduce build tooling, package.json, linters, or CI beyond the
  existing `.github/workflows/` (review + assistant workflows). The
  repo is intentionally plain Markdown.
- Don't add screenshots or binary assets. Routines should be text-only.
- Don't reformat the whole repo in an unrelated PR. Style fixes go in
  their own commit.

## Workflows in `.github/`

- `claude-code-review.yml` — runs on every PR opened by the repo owner,
  posts a structured review as a bot comment.
- `claude.yml` — handles `@claude` mentions in issues and PRs.

Both require the repo secret `ANTHROPIC_API_KEY` and a funded Anthropic
account. Don't edit these workflows casually — they reach out to the
Anthropic API and consume credits on every trigger.
