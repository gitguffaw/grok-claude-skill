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

Probe before assuming models, tools, MCP servers, plugins, or features. The installed binary wins over the bundled `~/.grok/*.md` docs.

## Model Resolution

Do not preserve a static default model in reusable launch patterns. Resolve at launch.

```bash
grok models
```

Inherit the local default when the user does not care which model runs. For the strongest run, pair the strongest available model with the highest supported effort:

```bash
grok -p "<prompt>" -m <MODEL> --effort <high|xhigh|max>
```

Use a fast model only when speed or cost outweighs capability, and only if `grok models` currently lists one (a `grok-composer-2.5-fast` has been seen intermittently). The available list is volatile — resolve it every time.

## Read-Only Analysis

```bash
grok -p "<question>" --tools "read_file,grep,list_dir"
grok -p "<question>" --tools "read_file,grep,list_dir,web_search,web_fetch" -m <MODEL> --effort high
```

## Bounded Execution

```bash
grok -p "<prompt>" --always-approve
grok -p "<prompt>" --always-approve --check
grok -p "<prompt>" -m <MODEL> --effort xhigh --always-approve --check
```

## Findings-Only Review

```bash
git diff | grok -p "Review for regressions, edge cases, and missing error handling. Findings only." --disallowed-tools "search_replace,run_terminal_cmd"
grok -p "Review the uncommitted changes. Findings only." --tools "read_file,grep,list_dir"
```

## Best-of-N

```bash
grok -p "<task>" --best-of-n 3 --always-approve
grok -p "<task>" --best-of-n 3 -m <MODEL> --effort high --always-approve --check
```

## Subagent Orchestration

```bash
grok -p "Use subagents. Spawn explore, then an implementer-persona agent, then a reviewer-persona agent. Task: <goal>." --always-approve
grok -p "<task>" --no-subagents
grok -p "<task>" --disallowed-tools "Agent(plan)"
```

Subagents are enabled by default and only spawn when the prompt explicitly asks.

## Multi-Turn / Machine-Readable Sessions

```bash
grok -p "<turn 1>" -s "<id>" --output-format json
grok -p "<turn 2>" -s "<id>" --output-format json | jq -r '.text'
SID=$(grok -p "<turn 1>" --output-format json | jq -r '.sessionId'); grok -p "<turn 2>" --resume "$SID"
grok -p "<next>" -c
```

`-s/--session-id` is documented in the bundled headless docs and accepted by the parser, but is not shown in `grok --help`; `--resume`/`-c` are the help-verifiable equivalents.

## Permission Posture

```bash
grok -p "<task>" --always-approve                          # unattended, trusted, well-scoped
grok -p "<task>" --allow "Bash(npm*)" --deny "Bash(sudo*)" # gate execution by rule
grok -p "<task>" --permission-mode plan                    # plan first, no edits
grok -p "<task>" --sandbox <PROFILE>                       # OS-level fs/network guardrails
```

## Determinism / Isolation

```bash
grok -p "<task>" --disable-web-search                      # no web variability
grok -p "<task>" -w <name>                                 # isolate edits in a worktree
grok -p "<task>" --rules "Never delete files. Keep public APIs stable."
```

## CI / Unattended

```bash
export XAI_API_KEY="xai-..."
grok -p "Review staged changes for obvious bugs. Reply OK or list issues." --always-approve --output-format json | jq -r '.text'
```
