# Grok Launch Patterns

Use these when Claude should drive inner Grok with explicit controls. Everything routes through headless `grok -p`.

## Preflight

```bash
grok --help
grok <cmd> --help
grok models
grok mcp list
grok plugin list
grok inspect
```

Probe before assuming models, tools, MCP servers, plugins, or features. Source of truth: binary help > user-guide > root `~/.grok/*.md` > skill.

## Model Resolution

Do not preserve a static default model in reusable launch patterns. Resolve at launch.

```bash
grok models
```

Inherit the local default when the user does not care which model runs. For the strongest run, pair the strongest available model with high reasoning effort:

```bash
grok -p "<prompt>" -m <MODEL> --reasoning-effort <high|xhigh|max>
```

Use a fast model only when speed or cost outweighs capability, and only if `grok models` currently lists one. The available list is volatile — resolve it every time.

## Read-Only Analysis

```bash
grok -p "<question>" --tools "read_file,grep,list_dir"
grok -p "<question>" --tools "read_file,grep,list_dir,web_search,web_fetch" -m <MODEL> --reasoning-effort high
```

## Bounded Execution

```bash
grok -p "<prompt>" --always-approve
grok -p "<prompt>" --always-approve --check
grok -p "<prompt>" -m <MODEL> --reasoning-effort xhigh --always-approve --check
```

## Findings-Only Review

Prefer read-only allowlist (proven no-write). Denylist form must include `write` or edits can still land via the separate `write` tool.

```bash
grok -p "Review for regressions, edge cases, and missing error handling. Findings only.

$(git diff)" --tools "read_file,grep,list_dir"
grok -p "Review the uncommitted changes. Findings only." --tools "read_file,grep,list_dir"
# secondary denylist (must include write):
grok -p "Review. Findings only.

$(git diff)" --disallowed-tools "search_replace,write,run_terminal_cmd"
```

## Best-of-N

```bash
grok -p "<task>" --best-of-n 3 --always-approve
grok -p "<task>" --best-of-n 3 -m <MODEL> --reasoning-effort high --always-approve --check
```

## Subagent Orchestration

```bash
grok -p "Use subagents. Spawn explore, then an implementer-persona agent, then a reviewer-persona agent. Task: <goal>." --always-approve
grok -p "<task>" --no-subagents
grok -p "<task>" --disallowed-tools "Agent(plan)"
```

Subagents are enabled by default and only spawn when the prompt explicitly asks.

## Multi-Turn / Machine-Readable Sessions

Preferred: capture auto session id, then resume.

```bash
SID=$(grok -p "<turn 1>" --output-format json --always-approve | jq -r '.sessionId')
grok -p "<turn 2>" --resume "$SID" --output-format json --always-approve | jq -r '.text'
grok -p "<next>" -c --output-format json
```

Optional create-only UUID (must be valid UUID; must not exist). Later turns always use `--resume` / `-c` — never reuse `-s`.

```bash
UUID=$(uuidgen | tr '[:upper:]' '[:lower:]')
grok -p "<turn 1>" -s "$UUID" --output-format json
grok -p "<turn 2>" --resume "$UUID" --output-format json
```

Fork on resume (optional new UUID via `-s`):

```bash
FORK=$(uuidgen | tr '[:upper:]' '[:lower:]')
grok -p "<branch>" --resume "$SID" --fork-session -s "$FORK" --output-format json
```

Structured output:

```bash
grok -p "<task>" --json-schema '{"type":"object","properties":{"ok":{"type":"boolean"}},"required":["ok"]}'
```

Streaming:

```bash
grok -p "<task>" --output-format streaming-json
```

## Permission Posture

```bash
grok -p "<task>" --always-approve                          # unattended, trusted, well-scoped
grok -p "<task>" --allow "Bash(npm*)" --deny "Bash(sudo*)" # gate execution by rule
grok -p "<task>" --permission-mode plan                    # enum accepted; headless plan enforcement NOT proven — prefer prompt + read-only --tools
grok -p "<task>" --sandbox <PROFILE>                       # OS-level fs/network guardrails
```

`--permission-mode plan` is listable from help but other modes (except `bypassPermissions` / always-approve / `default` wiring) are often accepted without full CLI enforcement. Do not treat `plan` alone as plan-first / no-edits in headless; use prompt discipline + `--tools "read_file,grep,list_dir"` (and avoid `--always-approve`) when edits must not happen.

## Determinism / Isolation

```bash
grok -p "<task>" --disable-web-search                      # disables web_search/web_fetch only; X search + image tools may remain
grok -p "<task>" --tools "read_file,grep,list_dir"         # stronger isolation: tight allowlist
grok -p "<task>" -w <name>                                 # isolate edits in a worktree
grok -p "<task>" -w <name> --worktree-ref <ref>            # base worktree on branch/tag/commit
grok -p "<task>" --rules "Never delete files. Keep public APIs stable."
```

## CI / Unattended

```bash
export XAI_API_KEY="xai-..."
grok -p "Review staged changes for obvious bugs. Reply OK or list issues.

$(git diff --staged)" --always-approve --output-format json | jq -r '.text'
```
