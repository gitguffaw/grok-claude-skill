# Parallel

## ENTER

- The work splits cleanly across roles (research, implement, review) that can run as inner Grok subagents, OR
- One shot is risky and N independent attempts improve the odds (host-side separate `grok -p` runs, or inner independent subagents).

## AVOID

- A single bounded run is enough: use `Exec`.
- Read-only investigation is enough: use `Analyze`.

## TWO FORMS

1. Subagent orchestration: let Grok spawn `general-purpose` / `explore` / `plan` children, optionally with personas.
2. Independent attempts: launch N separate `grok -p` processes from the host, then compare. There is no `--best-of-n` flag.

## MODIFIERS

- `Model`
- `Effort`
- `Permissions`

## DEFAULT LAUNCH

Subagent orchestration (subagents are on by default; ask for lanes in the prompt):

```bash
grok -p "Use subagents. Spawn an explore agent to map the module, then an implementer-persona agent to make the change, then a reviewer-persona agent to critique it. Task: <goal>." --always-approve
```

Model and effort:

```bash
grok -p "Use subagents. <lanes>. Task: <goal>." -m <MODEL> --reasoning-effort <high|xhigh> --always-approve
```

Constrain or disable subagents:

```bash
grok -p "<task>" --disallowed-tools "Agent(plan)"   # block one type
grok -p "<task>" --no-subagents                       # disable spawning
```

Independent host-side attempts (compare after):

```bash
grok -p "<task>" --always-approve
# launch additional independent grok -p processes with the same prompt
```

## PROMPT SHAPE

For subagent orchestration, make the lanes explicit:

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

Independent attempts reuse the `Exec` goal / scope / validation shape; each host-side run gets the same prompt.

## POST

- Read the synthesized or compared result.
- Confirm the kept variant actually meets the acceptance criteria.
- Inspect `git status` / `git diff` if edits were made.

## EXIT

- Accepted result.
- Or re-route to `Exec` / `Review`.

## Examples

```bash
grok -p "Use subagents. Spawn explore to map the HTTP client, then an implementer-persona agent to add retry-with-backoff without changing the public signature, then a reviewer-persona agent to critique tests and edge cases." --always-approve
```

## Current Facts

- There is no `--best-of-n` flag (parser rejects it).
- Subagents are enabled by default. Ask for them in the prompt when you want role-split work; disable with `--no-subagents`. The parent can also spawn on its own.
- Built-in subagent types: `general-purpose`, `explore`, `plan`. Bundled personas: `implementer`, `reviewer`, `researcher`, `test-writer`, `security-auditor`, `design-doc-writer`, `design-doc-reviewer`.
- Gate subagents with `--disallowed-tools "Agent"`, `Agent(explore)`, or `Agent(explore, plan)`.
