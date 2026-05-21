---
name: release-checklist
purpose: Pre-release verification before tagging or publishing.
inputs:
  - Target version (e.g. v1.4.0)
  - "Branch to release from (default: main)"
when-to-use: Immediately before cutting a release tag.
---

# Release Checklist

Verify the project at `{{branch}}` is ready to release as `{{version}}`.

## Steps

1. **Clean tree.** `git status` shows no uncommitted changes.
2. **Up to date.** Local branch matches `origin/{{branch}}`.
3. **Tests pass.** Run the full suite, not a subset.
4. **Type check & lint** pass with no new warnings.
5. **Changelog updated** for the new version, dated today.
6. **Version bumped** in the canonical place (package.json,
   pyproject.toml, Cargo.toml, etc.).
7. **No `TODO(release)` or `XXX` markers** introduced since the last
   tag.
8. **Dependencies:** no unintended upgrades; lockfile is consistent.
9. **Smoke test the built artifact** (binary, container, package) if
   the project produces one.

## Output format

A checklist with pass / fail / skipped for each item, followed by a
go / no-go recommendation.

## Guardrails

- Do **not** create the tag or push. Report status only — the human
  decides whether to ship.
- If any check is ambiguous, mark it `skipped` and explain why rather
  than guessing.
