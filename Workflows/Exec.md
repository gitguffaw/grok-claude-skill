# Exec

## ENTER

- Deliverable is concrete.
- Task is bounded.
- Acceptance criteria are known.
- Claude can inspect the result after the run.

## AVOID

- Unknowns still dominate: use `Analyze`.
- Only a review report is needed: use `Review`.
- One shot is risky and N independent attempts would help: use `Parallel` with `--best-of-n`.

## MODIFIERS

- `Model`
- `Effort`
- `SelfCheck`
- `Permissions`
- `ToolScope`
- `MaxTurns`
- `Worktree`

## DEFAULT LAUNCH

Bounded, unattended write run:

```bash
grok -p "<prompt>" --always-approve
```

Model and effort override:

```bash
grok -p "<prompt>" -m <MODEL> --effort <high|xhigh|max> --always-approve
```

Add a self-verification pass:

```bash
grok -p "<prompt>" --always-approve --check
```

Isolate file changes in a worktree:

```bash
grok -p "<prompt>" --always-approve -w <name>
```

Resolve `<MODEL>` from user intent, local config, or `grok models`; do not reuse stale model IDs.

## PROMPT SHAPE

```text
Goal: <required change>
Scope: <files/directories/subsystem>
Constraints:
- <constraint>
Validation:
- <test/lint/typecheck/manual check>
Non-goals:
- <explicitly excluded work>
```

## POST

- Read Grok's final message (or `--output-format json` `.text`).
- Inspect `git status`.
- Inspect `git diff`.
- Run requested validation locally if Grok did not.

## EXIT

- Accepted diff/result.
- Or re-route to:
  - `Analyze`
  - `Review`
  - `Parallel`

## Examples

```bash
grok -p "Goal: add input validation to src/forms/Register.tsx. Scope: registration form only. Constraints: keep the public API unchanged. Validation: pnpm test, pnpm lint. Non-goals: redesign the form UX." -m <MODEL> --effort high --always-approve --check
```

## Current Facts

- `--always-approve` auto-approves all tool executions; scope the prompt tightly.
- `--check`, `--effort`, `--max-turns`, `--tools`, and `--disallowed-tools` are headless-only.
- Web search (`web_search`/`web_fetch`) is on by default; add `--disable-web-search` for deterministic or offline runs.
- `-w/--worktree` runs in a new git worktree; review and merge the worktree afterward.
