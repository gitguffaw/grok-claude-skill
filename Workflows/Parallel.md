# Parallel

## ENTER

- One shot is risky and N independent attempts improve the odds, OR
- The work splits cleanly across roles (research, implement, review) that can run as subagents.

## AVOID

- A single bounded run is enough: use `Exec`.
- Read-only investigation is enough: use `Analyze`.

## TWO FORMS

1. Best-of-N: run the same task N ways in parallel and keep the best (headless-only).
2. Subagent orchestration: let Grok spawn `general-purpose` / `explore` / `plan` children, optionally with personas.

## MODIFIERS

- `Model`
- `Effort`
- `SelfCheck`
- `Permissions`

## DEFAULT LAUNCH

Best-of-N selection:

```bash
grok -p "<task>" --best-of-n <N> --always-approve
```

Best-of-N with model and effort:

```bash
grok -p "<task>" --best-of-n <N> -m <MODEL> --effort <high|xhigh|max> --always-approve
```

Subagent orchestration (subagents are on by default; ask for them explicitly):

```bash
grok -p "Use subagents. Spawn an explore agent to map the module, then an implementer-persona agent to make the change, then a reviewer-persona agent to critique it. Task: <goal>." --always-approve
```

Constrain or disable subagents:

```bash
grok -p "<task>" --disallowed-tools "Agent(plan)"   # block one type
grok -p "<task>" --no-subagents                       # disable spawning
```

## PROMPT SHAPE

Best-of-N reuses the `Exec` goal / scope / validation shape; each of the N attempts gets the same prompt. For subagent orchestration, make the lanes explicit:

```text
Use subagents.
Task: <goal>
Lanes:
- explore: <what to map>
- implementer persona: <change to make>
- reviewer persona: <what to critique>
Merge:
- one result, deduped, prioritized by impact
Constraints:
- <constraint>
```

## POST

- Read the selected or synthesized result.
- For best-of-N, confirm the kept variant actually meets the acceptance criteria.
- Inspect `git status` / `git diff` if edits were made.

## EXIT

- Accepted result.
- Or re-route to `Exec` / `Review`.

## Examples

```bash
grok -p "Implement a retry-with-backoff wrapper for the HTTP client. Keep the public signature. Add tests." --best-of-n 3 --always-approve --check
```

## Current Facts

- `--best-of-n <N>` is headless-only and runs N attempts in parallel before picking one.
- Subagents are enabled by default and only spawn when the prompt explicitly asks for them.
- Built-in subagent types: `general-purpose`, `explore`, `plan`. Personas: `implementer`, `reviewer`, `researcher`, `test-writer`, `security-auditor`, `design-doc-writer`, `design-doc-reviewer`.
- Gate subagents with `--disallowed-tools "Agent"`, `Agent(explore)`, or `Agent(explore, plan)`.
