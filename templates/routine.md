---
name: <short-name>
purpose: <one sentence describing what this routine does>
inputs:
  - <thing the user must provide, e.g. PR number, file path>
when-to-use: <when this routine is the right tool>
---

# <Title>

<One-paragraph description of what Claude should do when this routine runs.>

## Steps

1. <First step — be specific about what to read, run, or check.>
2. <Second step.>
3. <Third step.>

## Output format

<Describe the exact shape of the response you want — bullet list, table,
diff, etc. Be strict; this is what makes a routine reusable.>

## Guardrails

- <Things the routine must NOT do, e.g. "do not push", "do not edit
  production config".>
- <Stop conditions, e.g. "ask before deleting files".>
