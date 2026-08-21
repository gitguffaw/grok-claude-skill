---
name: Grok
description: Drive the local Grok CLI (xAI) as an inner coding and analysis agent through headless `grok -p`. Trigger when a host should delegate to local Grok instead of its own tools — analyze, implement, review, subagent orchestration, or a multi-turn session.
---

# Grok

Route bounded work through `grok -p`. Outer-host tools and inner Grok tools are different layers. If the task names Grok web search, subagents, personas, MCP, or plugins, satisfy that inside the launched run.

When model, effort, tools, MCP, or plugins matter, probe first: `grok --help`, `grok models`, `grok inspect`. Flag surface: `references/QuickRef.md`. Launch lines: `references/LaunchPatterns.md`.

## Modes

- `Analyze` — repo-grounded unknowns, no edits: `Workflows/Analyze.md`
- `Exec` — bounded change, unattended: `Workflows/Exec.md`
- `Review` — findings only: `Workflows/Review.md`
- `Parallel` — subagent and persona lanes: `Workflows/Parallel.md`
- `Session` — multi-turn via JSON `sessionId` → `--resume` / `-c`: `Workflows/Session.md`

## Router

- Repo unknowns, no edits → Analyze with `--tools "read_file,grep,list_dir"`
- Bounded deliverable → Exec with `--always-approve`
- Findings only → Review with the same read-only allowlist (denylist must include `write`)
- Role-split work → Parallel; ask for subagents in the prompt
- Independent attempts → N separate `grok -p` runs
- Work spans calls → Session; capture `sessionId`, continue with `--resume` / `-c`

## Launch

```bash
grok -p "<prompt>"
```

Add `--always-approve` when edits or commands must finish unattended. Add `--output-format json` to capture `sessionId`. Do not drive the interactive TUI from the host.

Inspect the result (diff, findings, or JSON `.text`) when the run returns.
