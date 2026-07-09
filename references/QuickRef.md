# Grok CLI Quick Reference

LAST VERIFIED:
- `grok 0.2.93 (f00f96316d4b) [stable]`
- `2026-07-08`

SOURCE OF TRUTH:
1. Installed `grok` binary (`grok --help`, live parse/runtime errors)
2. `~/.grok/docs/user-guide/`
3. Root `~/.grok/*.md` (often stale — do not copy session/effort/yolo claims)
4. This skill

PREFLIGHT:
- `grok --help`
- `grok <cmd> --help`
- `grok models`
- `grok mcp list`
- `grok plugin list`
- `grok inspect` / `grok inspect --json`
- `~/.grok/config.toml`

LAST OBSERVED LOCAL STATE (point-in-time, not contract):
- default model: `grok-4.5`
- also seen: `grok-composer-2.5-fast`
- catalog is volatile — resolve live every time; never pin a non-default model
- auth: logged in with grok.com; `XAI_API_KEY` for headless/CI
- native `mcp`/`plugin list` may look empty while `grok inspect` discovers Claude-compatible MCP/plugins

MODEL FACTS:
- resolve the live catalog with `grok models`; do not hard-code a permanent default in this skill
- set a persistent default with `[models] default = "<id>"` in `~/.grok/config.toml`
- override per run with `-m <MODEL>`

HEADLESS SPINE (`grok -p` / `--single`):
- `-p, --single <PROMPT>` single-turn headless; prints to stdout and exits
- `--prompt-file <PATH>` / `--prompt-json <JSON>` prompt from file / JSON content blocks
- `-m, --model <MODEL>`
- `--reasoning-effort <EFFORT>` (aliases: `--effort`) — levels include `none`, `minimal`, `low`, `medium`, `high`, `xhigh`, `max` (`max` aliases `xhigh` per user-guide; or a model menu option id); verify with help / rejection messages
- `--output-format <plain|json|streaming-json>` (headless mode; default plain)
- `--json-schema <SCHEMA>` structured output; implies `--output-format json`
- `--always-approve` auto-approve all tool executions (prefer this; a historical short CLI alias is accepted by the parser but not help-listed — do not use in recipes; `/yolo` is TUI-only)
- `--permission-mode <default|acceptEdits|auto|dontAsk|bypassPermissions|plan>` — enum from help; do not assume non-bypass modes fully enforce in headless CLI (user-guide wiring note)
- `--allow <RULE>` / `--deny <RULE>` `ToolPrefix(glob)` rules; repeatable; deny wins
- `--sandbox <PROFILE>` OS-level fs/network guardrails (env `GROK_SANDBOX`; profiles commonly include off, workspace, devbox, read-only, strict — verify live)
- `--tools <ids>` allowlist / `--disallowed-tools <ids>` denylist (headless automation)
- `--max-turns <N>` (headless automation)
- `--check` append self-verification loop (headless only; hidden alias `--self-verify` also accepted)
- `--best-of-n <N>` run N ways in parallel, pick best (headless only)
- `-s, --session-id <UUID>` **create-only** new session UUID (must be valid UUID; must not already exist). With `--resume`/`--continue`, only valid with `--fork-session`. Does **not** resume. Documented in `grok --help`.
- `-r, --resume [<ID>]` resume by id (or most recent if omitted); errors if missing when ID given and absent
- `-c, --continue` continue most recent session for cwd
- `--fork-session` when resuming/continuing, create a new session ID instead of reusing (optionally name via `-s`)
- `--restore-code` check out the original session's commit when resuming
- `--worktree-ref <REF>` / `--ref <REF>` branch/tag/commit base for `-w/--worktree`
- `--leader-socket <PATH>` share/isolate the backend leader process (root surface)
- `--leader` / `--no-leader` — agent-primary / help-hidden on root `-p` (documented on `grok agent --help`; accepted as hidden aliases on root)
- `--cwd <PATH>` working directory (session store is keyed by cwd under `~/.grok/sessions/`)
- `--rules <TEXT>` append to system prompt
- `--system-prompt-override <TEXT>` replace system prompt
- `--verbatim` send the prompt exactly as given
- `--disable-web-search` disables web_search/web_fetch only (on by default); X search and other tools may remain — prefer tight `--tools` for isolation
- `--no-subagents` / `--agent <NAME|path>` / `--agents <JSON>`
- `-w, --worktree [<NAME>]` run in a new git worktree
- `--experimental-memory` / `--no-memory`
- `--no-plan` disable plan mode
- Compaction (may be **hidden** from `grok --help`; binary accepts): `--compaction-mode <summary|transcript|segments>` / `--compaction-detail <none|minimal|balanced|verbose>` (env `GROK_COMPACTION_MODE` / `GROK_COMPACTION_DETAIL`)

HEADLESS-ONLY / HEADLESS AUTOMATION (help-marked or user-guide headless set):
- help-marked headless only: `--best-of-n`, `--check`; `--output-format` is for headless mode
- headless automation (user-guide): `--tools`, `--disallowed-tools`, `--max-turns`, `--agents`
- do **not** treat `--reasoning-effort`/`--effort`, `--permission-mode`, or `-s/--session-id` as headless-only unless live help marks them so

SESSION SEMANTICS (PROVEN):
- Multi-turn: `SID=$(… --output-format json | jq -r .sessionId)` then `grok -p … --resume "$SID"` or `-c`
- `-s` create-only UUID; reuse → error “already in use”; non-UUID → error
- Fork: `--resume $SID --fork-session` (+ optional `-s FORK_UUID`)
- Never teach create-or-resume with `-s`

DRIFT TO REMEMBER (binary vs docs):
- prefer `--always-approve`; a historical short CLI alias is accepted by the parser but not help-listed — do not use in recipes; `/yolo` is TUI-only
- full effort set is broader than old root docs (`none`/`minimal`/`xhigh`/`max` included; `max` aliases `xhigh`)
- root `~/.grok/*.md` still may claim create-or-resume `-s` and free-form session names — **stale**
- web search is ON by default in headless; `--disable-web-search` removes web_search/web_fetch only (not full external isolation)

OUTPUT FORMATS:
- `plain` (default): human-readable text
- `json`: object with `text`, `stopReason`, `sessionId`, `requestId`; may include `thought`; with `--json-schema` may include `structuredOutput`
- `streaming-json`: NDJSON events; `text` / `thought` events carry a `data` field; terminal `end` event carries `stopReason` / `sessionId` / `requestId` (event set may be non-exhaustive)
- extract: `grok -p "..." --output-format json | jq -r '.sessionId'`

BUILT-IN TOOL IDS (`--tools` / `--disallowed-tools`):
- Non-exhaustive catalog — prefer allowlist for read-only runs
- Proven/common: `read_file`, `search_replace`, `write`, `grep`, `list_dir`, `run_terminal_cmd` (shell — not `bash`; model-facing name may appear as `run_terminal_command`), `web_search`, `web_fetch`, `todo_write`
- Edit paths: `search_replace` **and** `write` (denylist without `write` does not block overwrites)
- `task`: unverified / do not rely as a first-class allowlist ID (live list may show `spawn_subagent` instead)
- `memory_search` exists but is a model-invokable memory tool gated by `--experimental-memory`, not a documented `--tools` allowlist entry
- subagent gates: `Agent`, `Agent(explore)`, `Agent(explore, plan)`

PERMISSION RULE PREFIXES (`--allow` / `--deny`):
- `Bash(...)`, `Edit(...)`, `Write(...)`, `Read(...)`, `Grep(...)`, `WebFetch(... | domain:host)`, `MCPTool(...)`
- globs: `*` single-level, `**` recursive; bare prefix matches all; deny precedence

SUBAGENTS / PERSONAS:
- types: `general-purpose` (all), `explore` (read-only), `plan`
- personas: `implementer`, `reviewer`, `researcher`, `test-writer`, `security-auditor`, `design-doc-writer`, `design-doc-reviewer`
- enabled by default; only spawn when the prompt asks; capability modes `read-only` | `read-write` | `execute` | `all`

SECONDARY SURFACES (not one-shot delegation):
- `grok agent stdio` ACP over stdin/stdout (IDE clients)
- `grok agent serve --bind <addr> --secret <tok>` / `grok agent headless --grok-ws-url <wss>` WebSocket
- `grok sessions list|search|delete` (subcommand required), `grok export <id> [out]`, `grok import [targets] --list --json`
- `grok mcp list|add|remove|doctor`, `grok plugin list|install|uninstall|update|enable|disable|details|validate|tag|marketplace`
- `grok worktree list|show|rm|gc|db`, `grok memory clear`, `grok inspect [--json]`

AUTH:
- browser: `grok login` (`--oauth`, `--device-auth`); stored in `~/.grok/auth.json`
- headless/CI: `export XAI_API_KEY="xai-..."` (takes precedence)

ENV:
- `XAI_API_KEY`, `GROK_SANDBOX`, `GROK_LOG_FILE`, `GROK_LOG_FILTER`, `RUST_LOG`
- compaction: `GROK_COMPACTION_MODE`, `GROK_COMPACTION_DETAIL`
- Do not assume `GROK_HOME` isolation without re-probe (not confirmed as supported on this binary).

EXIT CODES:
- `0` success; `1` error (auth, network, runtime, invalid session flags)

MACHINE SNAPSHOT:
- Flag surface capture: `references/cli-surface.json` (optional; regenerate via local harness when binary updates)
