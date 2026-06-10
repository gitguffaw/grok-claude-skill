# Grok CLI Quick Reference

LAST VERIFIED:
- `grok 0.2.39 (55a20b703aa) [stable]`
- `2026-06-10`

SOURCE OF TRUTH:
- The installed `grok` binary wins over bundled `~/.grok/*.md` docs when they disagree.
- Re-derive facts with `grok --help` and `grok <cmd> --help`.

PREFLIGHT:
- `grok --help`
- `grok <cmd> --help`
- `grok models`
- `grok mcp list`
- `grok plugin list`
- `grok inspect` / `grok inspect --json`
- `~/.grok/config.toml`

LAST OBSERVED LOCAL STATE:
- default model: `grok-build` (steady)
- available models: volatile — `grok-composer-2.5-fast` appeared in one `grok models` call and was gone minutes later; always resolve live, never pin a non-default model
- auth: logged in with grok.com; `XAI_API_KEY` for headless/CI
- `[ui] permission_mode = "always-approve"`, `yolo = false`, `fork_secondary_model = "grok-build"`
- native config empty: `grok mcp list` = no servers, `grok plugin list` = none
- but `grok inspect` discovers a Claude-compatible surface (e.g. `context7` MCP + many `~/.claude` plugins); run `grok inspect` before declaring a capability unavailable

MODEL FACTS:
- new sessions default to `grok-build`
- set a persistent default with `[models] default = "<id>"` in `~/.grok/config.toml`
- resolve the live catalog with `grok models`; do not hard-code a permanent default in this skill
- the available list is volatile (resolve every time); only `grok-build` has been steady
- override per run with `-m <MODEL>`

HEADLESS SPINE (`grok -p` / `--single`):
- `-p, --single <PROMPT>` single-turn headless; prints to stdout and exits
- `--prompt-file <PATH>` / `--prompt-json <JSON>` prompt from file / JSON content blocks
- `-m, --model <MODEL>`
- `--effort <low|medium|high|xhigh|max>` (headless-only)
- `--reasoning-effort <EFFORT>` reasoning-model knob (also accepted by `grok agent`)
- `--output-format <plain|json|streaming-json>`
- `--always-approve` auto-approve all tool executions
- `--permission-mode <default|acceptEdits|auto|dontAsk|bypassPermissions|plan>` (headless-only)
- `--allow <RULE>` / `--deny <RULE>` `ToolPrefix(glob)` rules; repeatable; deny wins (both modes)
- `--sandbox <PROFILE>` OS-level fs/network guardrails (env `GROK_SANDBOX`)
- `--tools <ids>` allowlist / `--disallowed-tools <ids>` denylist (headless-only)
- `--max-turns <N>` (headless-only)
- `--check` append self-verification loop (headless-only)
- `--best-of-n <N>` run N ways in parallel, pick best (headless-only)
- `-s, --session-id <ID>` create-or-resume named session (headless-only; accepted by the parser and documented in bundled headless docs, but NOT shown in `grok --help` — prefer `--resume`/`-c` for help-verifiable parity)
- `-r, --resume [<ID>]` resume by id (errors if missing)
- `-c, --continue` continue most recent session for cwd
- `--restore-code` check out the original session's commit when resuming
- `--compaction-mode <summary|transcript|segments>` / `--compaction-detail <none|minimal|balanced|verbose>` long-run context compaction (env `GROK_COMPACTION_MODE` / `GROK_COMPACTION_DETAIL`)
- `--leader` / `--no-leader` / `--leader-socket <PATH>` share or isolate the backend leader process (matters when running several Grok invocations at once)
- `--cwd <PATH>` working directory
- `--rules <TEXT>` append to system prompt
- `--system-prompt-override <TEXT>` replace system prompt
- `--verbatim` send the prompt exactly as given
- `--disable-web-search` remove web_search/web_fetch (on by default)
- `--no-subagents` / `--agent <NAME|path>` / `--agents <JSON>`
- `-w, --worktree [<NAME>]` run in a new git worktree
- `--experimental-memory` / `--no-memory`
- `--no-plan` disable plan mode

HEADLESS-ONLY FLAGS (ignored with a warning in the TUI):
- `--tools`, `--disallowed-tools`, `--max-turns`, `--effort`, `--permission-mode`, `-s/--session-id`, `--check`, `--best-of-n`

DRIFT TO REMEMBER (binary vs bundled docs):
- auto-approve flag is `--always-approve` (a bundled headless doc says `--yolo`; `/yolo` is only a TUI slash alias)
- `--effort` accepts five levels here (`low|medium|high|xhigh|max`); a bundled doc lists only three
- web search is ON by default in headless; disable with `--disable-web-search`

OUTPUT FORMATS:
- `plain` (default): human-readable text
- `json`: a single object `{ "text", "stopReason", "sessionId", "requestId" }` (the `.text` field exists only here)
- `streaming-json`: NDJSON events; `text` / `thought` events carry a `data` field, and only the terminal `end` event carries `stopReason` / `sessionId` / `requestId`
- extract: `grok -p "..." --output-format json | jq -r '.sessionId'`

BUILT-IN TOOL IDS (`--tools` / `--disallowed-tools`):
- `read_file`, `search_replace`, `grep`, `list_dir`, `run_terminal_cmd` (bash), `web_search`, `web_fetch`, `todo_write`, `task`
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
- `grok sessions list|search`, `grok export <id> [out]`, `grok import [targets] --list --json`
- `grok mcp list|add|remove|doctor`, `grok plugin list|install|uninstall|update|enable|disable|details|validate|tag|marketplace`
- `grok worktree list|show|rm|gc|db`, `grok memory clear`, `grok inspect [--json]`

AUTH:
- browser: `grok login` (`--oauth`, `--device-auth`); stored in `~/.grok/auth.json` (7-day expiry)
- headless/CI: `export XAI_API_KEY="xai-..."` (takes precedence)

ENV:
- `XAI_API_KEY`, `GROK_HOME`, `GROK_SANDBOX`, `GROK_LOG_FILE`, `GROK_LOG_FILTER`, `RUST_LOG`

EXIT CODES:
- `0` success; `1` error (auth, network, or runtime)
