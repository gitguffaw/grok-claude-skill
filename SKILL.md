---
name: Grok
description: Drive the local Grok CLI (xAI) as Claude's inner coding and analysis agent through headless `grok -p`, with explicit control over model, effort, web search, tool scope, permissions, sessions, and subagents/personas. Trigger when Claude Code, Codex, Grok, Antigravity, or another host should delegate to the local Grok CLI instead of relying only on the host's own tools.
---

# Grok

## Purpose

Use this skill to make a host agent operate Grok the way a strong human operator would: route every delegation through headless `grok -p`, pick the right outcome mode, set model, effort, tool scope, and permission posture explicitly when they matter, and direct inner Grok to use its own web search, subagents, and personas.

## Boundary

- Outer host and inner Grok are different tool layers.
- If the task says to use Grok web search, Grok subagents, Grok personas, Grok MCP, Grok plugins, or Grok-native tools, satisfy that inside the launched Grok run.
- Do not silently substitute Claude's own web tools or MCP tools when the request is explicitly about inner Grok capability.
- Use the host's own tools only for maintaining this skill, validating local Grok state, or when inner Grok capability is unavailable and that fallback is explicitly acceptable.

## Last Verified

- Skill package `v1.0.6`.
- Verified against `grok 1.0.5 (5115b46bc909) [stable]`.
- Verified on `2026-08-21`.
- Verified local auth: logged in with grok.com (browser session); `XAI_API_KEY` is the headless/CI path.
- Live-probed this pass: JSON `{text, stopReason, sessionId, requestId}` plus `thought` and spend fields; capture `sessionId` then `--resume` reused the same id; `-s` non-UUID errors; `--check` / `--best-of-n` rejected; `-w` with `-p` created no worktree; Analyze `--tools "read_file,grep,list_dir"` ran.

## Last Observed Local State

- Default model from `grok models` (point-in-time): `grok-4.6`
- Also seen: `grok-4.5`
- Model catalog is volatile. Resolve the live list every time; never pin a non-default model as reliably present.
- Live `--reasoning-effort` accept list from rejection message: `low`, `medium`, `high`, `xhigh`. User-guide lists additional tiers (`none`, `minimal`, `max`) that currently reject — trust the live error over docs.
- Built-in tool IDs for `--tools` / `--disallowed-tools` (non-exhaustive; prefer allowlist for read-only): `read_file`, `search_replace`, `write`, `grep`, `list_dir`, `run_terminal_cmd` (shell — not `bash`; model-facing name may appear as `run_terminal_command`), `web_search`, `web_fetch`, `todo_write`. Edit paths include both `search_replace` and **`write`**. CLI subagent gates: `--no-subagents` and `--disallowed-tools "Agent(...)"`. Inner spawn tool is `spawn_subagent` (not a documented `--tools` id). `memory_search` is a model-invokable memory tool gated by memory enablement (`GROK_MEMORY=1` or hidden `--experimental-memory`), not a documented `--tools` allowlist entry.
- Built-in subagent types: `general-purpose`, `explore`, `plan` (local plugin agents may add more — treat those as machine-local).
- Bundled personas: `implementer`, `reviewer`, `researcher`, `test-writer`, `security-auditor`, `design-doc-writer`, `design-doc-reviewer`
- Native `grok mcp list` / `grok plugin list` may look empty or sparse while `grok inspect` discovers Claude-compatible MCP/plugins. Prefer `grok inspect` before declaring a capability unavailable.
- Treat these as machine-local observations, not universal Grok contract.

## Preflight

When model choice, effort, tool availability, permission posture, or feature availability matters, probe current Grok state first.

- `grok --help`
- `grok <cmd> --help`
- `grok models`
- `grok mcp list`
- `grok plugin list`
- `grok inspect` (or `grok inspect --json`) — authoritative discovered config for the current directory
- `~/.grok/config.toml`
- Source of truth hierarchy: **binary help + live parse errors > `~/.grok/docs/user-guide/` > root `~/.grok/*.md` > this skill**. Root docs can lag badly; user-guide can still lag flag branding.

## Model Selection Policy

Do not encode a permanent default model in this skill. Grok model IDs and availability move over time and depend on account and runtime.

- If the user names a model, use it only after confirming it appears in `grok models`. If absent, say so and choose a nearby available model only when intent is clear.
- If the user does not name a model, inherit the local default (`[models] default` in config, or the runtime default from `grok models`). Do not hard-code an ID in launch examples.
- If the user asks for the strongest or highest-thinking Grok, resolve the model at launch from `grok models` and pair the strongest available model with a high `--reasoning-effort` (verify accepted levels with live parse errors).
- For fastest or lightweight work, prefer a fast model **only if `grok models` currently lists one** and only when the user prioritizes speed or cost over capability.
- Override explicitly with `-m <MODEL>`, and add `--reasoning-effort <LEVEL>` (alias: `--effort`) for reasoning depth.

## Primary Modes

- `Analyze`: reduce uncertainty, inspect repo facts, read-only investigation
- `Exec`: produce a change or deliverable, bounded and unattended
- `Review`: findings only, no edits
- `Parallel`: explicit subagent and persona orchestration (and host-side independent `grok -p` runs when N separate attempts help)
- `Session`: multi-turn, scripted, or machine-readable delegation across calls

## Modifiers

Apply zero or more modifiers to the primary mode.

- `Model`: `-m <MODEL>` after resolving the model from user intent, config, or `grok models`.
- `Effort`: `--reasoning-effort <LEVEL>` (alias `--effort`). Live accept list this binary: `low`, `medium`, `high`, `xhigh`. Verify with a rejection message before using any other tier. Prefer `high` or `xhigh` for intelligence-sensitive work. Not headless-only.
- `Machine`: `--output-format <json|streaming-json|streaming-messages-json>` for parseable output. `json` returns `{text, stopReason, sessionId, requestId}` and may include `thought` plus spend fields (`usage`, `num_turns`, `modelUsage`, cost) when the prompt reached the model. `streaming-json` is NDJSON of ACP session updates; terminal `end` carries metadata. `streaming-messages-json` is NDJSON in the Anthropic Messages API wire format; `--include-partial-messages` only affects that format. Parse with `jq`.
- `JsonSchema`: `--json-schema '<SCHEMA>'` constrains structured output and implies `--output-format json`.
- `WebSearch`: Grok's `web_search` and `web_fetch` are ON by default in headless. Do not "enable" search. `--disable-web-search` disables web_search/web_fetch only; X search and other tools may remain — use a tight `--tools` allowlist for stronger isolation.
- `Permissions`: Prefer help-visible `--always-approve` for unattended runs. A historical short CLI alias is accepted by the parser and listed in user-guide, but not in `grok --help` — do not use it in skill recipes. `/yolo` is TUI-only. Permission enum: `--permission-mode <default|acceptEdits|auto|dontAsk|bypassPermissions|plan>` (listable from help). Finer control via repeatable `--allow`/`--deny` `ToolPrefix(glob)` rules (deny wins); OS-level `--sandbox <PROFILE>`.
- `ToolScope`: `--tools <ids>` (allowlist) and `--disallowed-tools <ids>` (denylist, applied after `--tools`). Use internal tool IDs; catalog is non-exhaustive — prefer allowlist for read-only. User-guide: MCP meta-tools remain unless denied. `--disallowed-tools` also accepts `Agent`, `Agent(explore)`, `Agent(explore, plan)` to gate subagents. Headless automation knobs: tools / disallowed-tools / max-turns / agents.
- `Subagents`: enabled by default. `--agent <NAME|path>` selects an agent definition; `--agents <JSON>` supplies inline definitions; `--no-subagents` disables spawning; personas shape subagent behavior. Block specific types via `--disallowed-tools "Agent(...)"`.
- `MaxTurns`: `--max-turns <N>` bounds agentic turns (headless automation).
- `Plan`: Do **not** rely on `--permission-mode plan` alone for plan-first / no-edits in headless. Prefer prompt discipline + read-only `--tools` (and no `--always-approve`). Flag remains listable from help; `--no-plan` disables plan mode. TUI/`enter_plan_mode` is the stronger plan surface until flag enforcement is proven.
- `Worktree`: `-w [<NAME>]` creates a git worktree for **interactive** sessions. Help: headless (`-p`) does **not** create a worktree from this flag. For isolated headless edits, `--cwd` into an existing worktree, or ask inner Grok to spawn a subagent with worktree isolation. `--worktree-ref` / `--ref` sets branch/tag/commit base when a worktree is actually created.
- `Rules`: `--rules "<text>"` appends to the system prompt; `--system-prompt-override "<text>"` replaces it; `--verbatim` sends the prompt exactly as given.
- `SessionControl`: multi-turn via capture `sessionId` → `--resume` / `-c`. `--resume` accepts a session ID or a title for the current directory (scripts should pass IDs). Optional create-only `-s <UUID>` (must be valid UUID, must not exist). Fork with `--fork-session` on resume (optional `-s` names the fork UUID). `--restore-code` restores the original session's repository snapshot on resume (remote sessions require `--worktree`; without the flag, resume restores conversation only).

## Launch Rule

Mode expresses outcome shape. Modifiers decide the launch surface. For Claude-to-Grok delegation, the surface is almost always headless `grok -p`.

- Use `grok -p "<prompt>"` (`-p` is short for `--single`) for every bounded delegation: analysis, execution, review, and sessions.
- Add `--always-approve` when the run must complete unattended and edits or commands are expected.
- Add `--output-format json` whenever Claude needs to capture `sessionId` or parse the result.
- Reserve `grok agent stdio|serve|headless` for ACP, IDE, and WebSocket integration, not one-shot delegation.
- Do not drive the interactive `grok` TUI from Claude's Bash; it is a user-facing surface. Launch it for the user only when live human steering is the point.

## Router

- IF repo-grounded unknowns dominate THEN primary mode `Analyze` with a read-only `--tools` allowlist (bare `-p` only when unrestricted tools are intentional, e.g. pure web-external Q&A)
- IF the task is bounded and should produce a result or diff THEN primary mode `Exec` plus `--always-approve`
- IF output should be findings only THEN primary mode `Review` with `--tools "read_file,grep,list_dir"` (or denylist that includes `write`)
- IF current external information must be gathered THEN keep the mode and rely on Grok's default `web_search`/`web_fetch` (or allowlist them back if using `--tools`)
- IF the work splits across roles THEN primary mode `Parallel` with subagents and personas
- IF one shot is risky and N independent attempts help THEN launch N separate `grok -p` runs (host-side) or ask inner Grok to spawn independent subagents — there is no `--best-of-n` flag
- IF the work spans multiple calls or needs parseable output THEN primary mode `Session`: capture `sessionId` from JSON → `--resume "$SID"` or `-c` (optional create-only `-s <UUID>`, fork with `--fork-session`)
- IF planning should precede edits THEN prefer prompt discipline + read-only tools / no `--always-approve`; do not route solely to `--permission-mode plan` for headless enforcement

## Routing Bias

- Default to headless `grok -p`; it is the spine.
- Prefer `Exec` when the job is truly bounded and one-shot; put validation in the prompt (there is no `--check` flag).
- Prefer `Review` for critique without edits.
- Prefer `Parallel` when the work splits cleanly across inner subagent roles.
- Prefer Grok-native web search, subagents, and personas over outer-Claude substitutes when the user wants Grok capability specifically.
- Leave web search on unless the task needs isolation from web_search/web_fetch; then add `--disable-web-search` (X search may still remain — use tight `--tools` for stronger isolation).

## Claude Contract

- Specify the primary mode first.
- Add modifiers explicitly when they matter.
- Probe current Grok state before assuming models, MCP servers, plugins, tools, or features exist.
- Capture the `--output-format json` `sessionId` when the work may continue; resume with `--resume` / `-c`, never by reusing `-s`.
- Inspect the resulting diff, findings, or synthesis after Grok returns.
- Re-route if the task changes shape mid-stream.

## Hard Rules

- Source of truth: **binary > `~/.grok/docs/user-guide/` > root `~/.grok/*.md` > skill**. Trust `grok --help` and live parse errors over docs.
- Do not invent flags or model IDs. Resolve models via `grok models`.
- Do not pin model IDs in reusable examples or launch patterns; use `<MODEL>` placeholders. Observed-state sections may name a model only as a point-in-time observation.
- Prefer help-visible `--always-approve` for unattended auto-approval. A historical short CLI alias is accepted by the parser (not help-listed) — do not use it in skill recipes. `/yolo` is only a TUI slash alias for `/always-approve`.
- Prefer `--reasoning-effort`; `--effort` is its alias. Live accepted levels this binary: `low`, `medium`, `high`, `xhigh` — re-verify with rejection messages. Do not claim effort is headless-only.
- Headless-only (or headless automation) knobs include `--tools`, `--disallowed-tools`, `--max-turns`, `--agents` (and help-marked headless flags such as `--output-format`). Do **not** treat effort, permission-mode, or `-s` as headless-only without re-verification.
- There is no `--check` / `--self-verify` / `--best-of-n` on this binary (parser rejects them). Self-verification lives in the prompt. Independent attempts are N host-side `grok -p` runs or inner subagents.
- `-s/--session-id` is **create-only**: valid UUID, must not already exist. It is documented in `grok --help`. Reuse of an existing UUID **errors**. Non-UUID IDs **error**. Multi-turn: capture `sessionId` → `--resume` / `-c`. `--resume` may also match a session title in the current directory; scripts should pass IDs. With resume/continue, naming a new ID requires `--fork-session` (+ optional `-s`). Never teach create-or-resume via `-s`.
- Web search is ON by default in headless — never tell Grok to "enable" search; `--disable-web-search` turns off web_search/web_fetch only (X search may remain). Prefer tight `--tools` for isolation.
- Do not pass `-w` on `grok -p` expecting a new worktree; help says headless does not create one from that flag.
- Do not confuse outer Claude tools with inner Grok tools.
- Do not present machine-local observations (models, MCP, plugins) as universal Grok contract. Model availability is volatile; resolve with `grok models` and `grok inspect` before asserting availability.
- Do not drive the interactive TUI from Claude; use `grok -p`.
- Do not use `Review` when the expectation is immediate implementation.

## Secondary Surfaces

- `grok agent stdio`: ACP over stdin/stdout for IDE and editor clients
- `grok agent serve` / `grok agent headless`: WebSocket server / relay for networked or web clients
- `grok sessions list|search|delete`, `grok export <id>`: session inventory and transcript export
- `grok mcp`, `grok plugin`: manage Grok MCP endpoints and plugin/marketplace sources
- `grok worktree`, `grok memory`, `grok inspect`: worktree management, cross-session memory, config introspection
- `grok doctor`, `grok du` (`disk-usage`), `grok wrap`: terminal diagnostics, home-dir disk usage, clipboard wrap

Keep these available, but use them intentionally.

## Mode Index

- `Analyze`: `Workflows/Analyze.md`
- `Exec`: `Workflows/Exec.md`
- `Review`: `Workflows/Review.md`
- `Parallel`: `Workflows/Parallel.md`
- `Session`: `Workflows/Session.md`

## Modifier Reference

Modifiers have no workflow files of their own. Their launch lines live in `references/LaunchPatterns.md` (see all sections there: Model Resolution, Read-Only Analysis, Bounded Execution, Findings-Only Review, Subagent Orchestration, Independent Attempts, Multi-Turn / Machine-Readable Sessions, Permission Posture, Determinism / Isolation, CI / Unattended). `references/QuickRef.md` is the flag-by-flag surface.

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
Controls: --always-approve; put tests/lint in the prompt Validation section; resolve model + --reasoning-effort high
```

**Example 3**
```text
Task: "Have Grok review my uncommitted diff for bugs, findings only."
Route: Review
Controls: embed or --prompt-file the diff; --tools "read_file,grep,list_dir"
```

**Example 4**
```text
Task: "Let Grok explore the module, implement the change, then critique it."
Route: Parallel
Controls: --always-approve; prompt asks for explore then implementer-persona then reviewer-persona subagents
```

**Example 5**
```text
Task: "Run a two-step Grok review of this PR and parse the output in a script."
Route: Session
Controls: capture sessionId from --output-format json → --resume "$SID"; jq '.text' / '.sessionId'
```

## References

- `references/QuickRef.md`
- `references/LaunchPatterns.md`
- `Workflows/*.md`
