---
name: Grok
description: Drive the local Grok CLI (xAI) as Claude's inner coding and analysis agent through headless `grok -p`, with explicit control over model, effort, web search, tool scope, permissions, sessions, subagents/personas, best-of-N, and self-check. Trigger when Claude should delegate to Grok instead of relying only on Claude's own tools.
---

# Grok

## Purpose

Use this skill to make Claude operate Grok the way a strong human operator would: route every delegation through headless `grok -p`, pick the right outcome mode, set model, effort, tool scope, and permission posture explicitly when they matter, and direct inner Grok to use its own web search, subagents, personas, best-of-N, and self-check.

## Boundary

- Outer Claude and inner Grok are different tool layers.
- If the task says to use Grok web search, Grok subagents, Grok personas, Grok MCP, Grok plugins, or Grok-native tools, satisfy that inside the launched Grok run.
- Do not silently substitute Claude's own web tools or MCP tools when the request is explicitly about inner Grok capability.
- Use Claude's own tools only for maintaining this skill, validating local Grok state, or when inner Grok capability is unavailable and that fallback is explicitly acceptable.

## Last Verified

- Verified against `grok 0.2.39 (55a20b703aa) [stable]`.
- Verified on `2026-06-10`.
- Verified local auth: logged in with grok.com (browser session); `XAI_API_KEY` is the headless/CI path.

## Last Observed Local State

- Default model from `grok models`: `grok-build` (steady)
- Available models are volatile: `grok-composer-2.5-fast` appeared in one `grok models` call and was gone from another minutes later. Resolve the live list every time; never pin a non-default model as reliably present.
- `~/.grok/config.toml`: `[ui] permission_mode = "always-approve"`, `yolo = false`, `fork_secondary_model = "grok-build"`
- Native config is empty (`grok mcp list`: no servers; `grok plugin list`: none), but `grok inspect` discovers a Claude-compatible surface anyway — e.g. the `context7` MCP and dozens of `~/.claude` plugins. Run `grok inspect` (not just `mcp`/`plugin list`) before declaring a capability unavailable.
- Built-in tool IDs (for `--tools` / `--disallowed-tools`): `read_file`, `search_replace`, `grep`, `list_dir`, `run_terminal_cmd`, `web_search`, `web_fetch`, `todo_write`, `task`. `memory_search` is a model-invokable memory tool gated by `--experimental-memory`, not a documented `--tools` allowlist entry.
- Built-in subagent types: `general-purpose`, `explore`, `plan`
- Built-in personas: `implementer`, `reviewer`, `researcher`, `test-writer`, `security-auditor`, `design-doc-writer`, `design-doc-reviewer`
- Treat these as machine-local observations, not universal Grok contract.

## Preflight

When model choice, effort, tool availability, permission posture, or feature availability matters, probe current Grok state first.

- `grok --help`
- `grok <cmd> --help`
- `grok models`
- `grok mcp list`
- `grok plugin list`
- `grok inspect` (or `grok inspect --json`) — the authoritative view of what Grok actually discovers for the current directory: models, rules, skills, and Claude-compatible MCP servers and plugins. It surfaces capabilities that `grok mcp list` / `grok plugin list` do not.
- `~/.grok/config.toml`
- The installed `grok` binary is the source of truth. The bundled `~/.grok/*.md` docs can lag the binary — when they disagree, trust `grok --help`.

## Model Selection Policy

Do not encode a permanent default model in this skill. Grok model IDs and availability move over time and depend on account and runtime.

- If the user names a model, use it only after confirming it appears in `grok models`. If absent, say so and choose a nearby available model only when intent is clear.
- If the user does not name a model, inherit the local default (`[models] default` in config, or the runtime default — today `grok-build`). This lets config and future Grok migrations do their job.
- If the user asks for the strongest or highest-thinking Grok, resolve the model at launch from `grok models` and pair the strongest available model with the highest supported `--effort`.
- For fastest or lightweight work, prefer a fast model **only if `grok models` currently lists one** (a `grok-composer-2.5-fast` has been seen intermittently) and only when the user prioritizes speed or cost over capability. Do not assume any non-default model is present without checking.
- Override explicitly with `-m <MODEL>`, and add `--effort <LEVEL>` for reasoning depth.

## Primary Modes

- `Analyze`: reduce uncertainty, inspect repo facts, read-only investigation
- `Exec`: produce a change or deliverable, bounded and unattended
- `Review`: findings only, no edits
- `Parallel`: best-of-N selection and/or explicit subagent and persona orchestration
- `Session`: multi-turn, scripted, or machine-readable delegation across calls

## Modifiers

Apply zero or more modifiers to the primary mode.

- `Model`: `-m <MODEL>` after resolving the model from user intent, config, or `grok models`.
- `Effort`: `--effort <low|medium|high|xhigh|max>` (headless-only). Choose the highest level for max think; `high` is the floor for intelligence-sensitive work. `--reasoning-effort <EFFORT>` is the reasoning-model-specific knob and is also accepted by `grok agent`.
- `Machine`: `--output-format <json|streaming-json>` for parseable output. `json` returns `{text, stopReason, sessionId, requestId}`; `streaming-json` is NDJSON `text`/`thought`/`end` events. Parse with `jq`.
- `WebSearch`: Grok's `web_search` and `web_fetch` are ON by default in headless. Do not "enable" search. To turn it off, use `--disable-web-search`.
- `Permissions`: `--always-approve` for unattended runs; `--permission-mode <default|acceptEdits|auto|dontAsk|bypassPermissions|plan>`; finer control via repeatable `--allow`/`--deny` `ToolPrefix(glob)` rules (deny wins); OS-level `--sandbox <PROFILE>`.
- `ToolScope`: `--tools <ids>` (allowlist, disables default tool injection) and `--disallowed-tools <ids>` (denylist, applied after `--tools`). Use internal tool IDs. `--disallowed-tools` also accepts `Agent`, `Agent(explore)`, `Agent(explore, plan)` to gate subagents. Headless-only.
- `Subagents`: enabled by default. `--agent <NAME|path>` selects an agent definition; `--agents <JSON>` supplies inline definitions; `--no-subagents` disables spawning; personas shape subagent behavior. Block specific types via `--disallowed-tools "Agent(...)"`.
- `SelfCheck`: `--check` appends a self-verification loop (headless-only).
- `MaxTurns`: `--max-turns <N>` bounds agentic turns (headless-only).
- `Plan`: `--permission-mode plan` for plan-first work; `--no-plan` disables plan mode.
- `Worktree`: `-w [<NAME>]` runs the session in a new git worktree for isolation.
- `Rules`: `--rules "<text>"` appends to the system prompt; `--system-prompt-override "<text>"` replaces it; `--verbatim` sends the prompt exactly as given.

## Launch Rule

Mode expresses outcome shape. Modifiers decide the launch surface. For Claude-to-Grok delegation, the surface is almost always headless `grok -p`.

- Use `grok -p "<prompt>"` (`-p` is short for `--single`) for every bounded delegation: analysis, execution, review, and sessions.
- Add `--always-approve` when the run must complete unattended and edits or commands are expected.
- Add `--output-format json` whenever Claude needs to capture `sessionId` or parse the result.
- Reserve `grok agent stdio|serve|headless` for ACP, IDE, and WebSocket integration, not one-shot delegation.
- Do not drive the interactive `grok` TUI from Claude's Bash; it is a user-facing surface. Launch it for the user only when live human steering is the point.

## Router

- IF repo-grounded unknowns dominate THEN primary mode `Analyze`, optionally with a read-only `--tools` allowlist
- IF the task is bounded and should produce a result or diff THEN primary mode `Exec` plus `--always-approve`
- IF output should be findings only THEN primary mode `Review` with write and shell tools stripped
- IF current external information must be gathered THEN keep the mode and rely on Grok's default `web_search`/`web_fetch`
- IF one shot is risky and N independent attempts help THEN primary mode `Parallel` plus `--best-of-n <N>`
- IF Grok should split work across roles THEN primary mode `Parallel` with subagents and personas
- IF the work spans multiple calls or needs parseable output THEN primary mode `Session` plus a session flag (`--resume`/`-c`, or named `-s <id>`) and `--output-format json`
- IF planning should precede edits THEN add `Plan` (`--permission-mode plan`)

## Routing Bias

- Default to headless `grok -p`; it is the spine.
- Prefer `Exec` when the job is truly bounded and one-shot; add `--check` when correctness matters.
- Prefer `Review` for critique without edits.
- Prefer `Parallel` (`--best-of-n`) when one shot is risky and you can afford N attempts.
- Prefer Grok-native web search, subagents, and personas over outer-Claude substitutes when the user wants Grok capability specifically.
- Leave web search on unless the task needs determinism or isolation; then add `--disable-web-search`.

## Claude Contract

- Specify the primary mode first.
- Add modifiers explicitly when they matter.
- Probe current Grok state before assuming models, MCP servers, plugins, tools, or features exist.
- Capture the `--output-format json` `sessionId` when the work may continue.
- Inspect the resulting diff, findings, or synthesis after Grok returns.
- Re-route if the task changes shape mid-stream.

## Hard Rules

- The installed `grok` binary is the source of truth; trust `grok --help` over the bundled `~/.grok/*.md` docs when they disagree.
- Do not invent flags or model IDs. Resolve models via `grok models`.
- Do not pin model IDs in reusable examples or launch patterns; use `<MODEL>` placeholders. Observed-state sections may name a model only as a point-in-time observation.
- Use `--always-approve` (not the docs' historical `--yolo`) for unattended auto-approval; `/yolo` is only a TUI slash alias for `/always-approve`.
- `--effort` accepts `low|medium|high|xhigh|max` on this binary (a bundled doc still lists only three levels); verify with `grok --help`.
- `--tools`, `--disallowed-tools`, `--max-turns`, `--effort`, `--permission-mode`, `-s/--session-id`, `--check`, and `--best-of-n` are headless-only; they are ignored with a warning in the TUI.
- `-s/--session-id` works (the parser accepts it and the bundled headless docs document it) but is NOT shown in `grok --help`; for help-verifiable session control prefer `--resume <id>` / `-c`, and treat `-s` as the bundled-doc-recommended named-session path.
- Web search is ON by default in headless — never tell Grok to "enable" search; disable with `--disable-web-search` when needed.
- Do not confuse outer Claude tools with inner Grok tools.
- Do not present machine-local observations (models, MCP, plugins) as universal Grok contract. Model availability is volatile, and `grok mcp list`/`grok plugin list` understate what Grok discovers — resolve with `grok models` and `grok inspect` before asserting availability.
- Do not drive the interactive TUI from Claude; use `grok -p`.
- Do not use `Review` when the expectation is immediate implementation.
- Do not use `--best-of-n` or subagents unless N independent attempts or explicit role-splitting actually help.

## Secondary Surfaces

- `grok agent stdio`: ACP over stdin/stdout for IDE and editor clients
- `grok agent serve` / `grok agent headless`: WebSocket server / relay for networked or web clients
- `grok sessions list|search`, `grok export <id>`, `grok import`: session inventory and transcript movement
- `grok mcp`, `grok plugin`: manage Grok MCP endpoints and plugin/marketplace sources
- `grok worktree`, `grok memory`, `grok inspect`: worktree management, cross-session memory, config introspection

Keep these available, but use them intentionally.

## Mode Index

- `Analyze`: `Workflows/Analyze.md`
- `Exec`: `Workflows/Exec.md`
- `Review`: `Workflows/Review.md`
- `Parallel`: `Workflows/Parallel.md`
- `Session`: `Workflows/Session.md`

## Modifier Reference

Modifiers have no workflow files of their own. Their launch lines live in `references/LaunchPatterns.md` (Read-Only Analysis, Bounded Execution, Best-of-N, Subagent Orchestration, Multi-Turn / Machine-Readable Sessions, Permission Posture, Determinism / Isolation). `references/QuickRef.md` is the flag-by-flag surface.

## Examples

**Example 1**
```text
Task: "Use Grok to map how auth flows through this repo before we touch anything."
Route: Analyze
Controls: read-only --tools "read_file,grep,list_dir"; no approval needed
```

**Example 2**
```text
Task: "Implement the retry wrapper with Grok and have it verify its own work."
Route: Exec
Controls: --always-approve --check, resolve model + --effort high
```

**Example 3**
```text
Task: "Have Grok review my uncommitted diff for bugs, findings only."
Route: Review
Controls: git diff | grok -p ... --disallowed-tools "search_replace,run_terminal_cmd"
```

**Example 4**
```text
Task: "Let Grok try this refactor three ways and keep the best."
Route: Parallel
Controls: --best-of-n 3 --always-approve --check
```

**Example 5**
```text
Task: "Run a two-step Grok review of this PR and parse the output in a script."
Route: Session
Controls: -s "<id>" --output-format json, jq '.text' / '.sessionId'
```

## References

- `references/QuickRef.md`
- `references/LaunchPatterns.md`
- `Workflows/*.md`
