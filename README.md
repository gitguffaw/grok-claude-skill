# Grok skill

A portable **agent skill** that teaches an AI coding agent to drive the local **Grok CLI** (xAI) as an inner agent. Written in the standard `SKILL.md` format for **Claude Code**, **Codex**, and **Grok**.

When the host should delegate to Grok instead of using only its own tools, this skill routes through headless `grok -p` with explicit control over model, effort, web search, tool scope, permissions, sessions, subagents/personas, best-of-N, and self-check.

## Release stamp

| Field | Value |
|-------|--------|
| **Skill release** | `v0.2.93` (aligned to verified CLI) |
| **Verified binary** | `grok 0.2.93 (f00f96316d4b) [stable]` |
| **Verified on** | 2026-07-08 |
| **Changelog** | [`CHANGELOG-P0.md`](CHANGELOG-P0.md) |

**Breaking / critical corrections vs prior skill (0.2.39-era):**

- Multi-turn: capture `sessionId` from `--output-format json`, then **`--resume` / `-c`**. Do **not** reuse `-s` to continue.
- `-s/--session-id` is **create-only** (valid UUID, errors if already in use). Documented in `grok --help`.
- Review / findings-only: prefer `--tools "read_file,grep,list_dir"`. Denylist form must include **`write`**.
- Models: resolve live with `grok models` (observed default at verify time: `grok-4.5`).

## What it does

- **Spine:** headless `grok -p` for delegation. `grok agent` is ACP/IDE; interactive TUI is user-facing.
- **Modes:** `Analyze`, `Exec`, `Review`, `Parallel`, `Session`.
- **Modifiers:** model, effort (`--reasoning-effort` / `--effort`), output format, `--json-schema`, permissions, tool scope, self-check, worktree, rules, fork-session.
- **Guardrails:** installed binary beats bundled docs; models never pinned in recipes; `grok inspect` for discovered capabilities.

## Requirements

- **`grok` CLI** installed and authenticated (`grok login`, or `XAI_API_KEY` for headless/CI). See <https://x.ai/cli>.
- Host that loads `SKILL.md` skills: **Claude Code**, **Codex**, or **Grok**.
- This release was verified against **`grok 0.2.93`**. After upgrading Grok, re-check `grok --version` against the stamp in `SKILL.md`.

## Install

Install under the name **`Grok`** (TitleCase).

### Claude Code

```bash
# Personal — every project
git clone https://github.com/gitguffaw/grok-claude-skill ~/.claude/skills/Grok

# Project-scoped
git clone https://github.com/gitguffaw/grok-claude-skill <your-repo>/.claude/skills/Grok
```

Start a new Claude session after install. Nudge with: *"use the Grok skill to …"*.

### Codex

```bash
git clone https://github.com/gitguffaw/grok-claude-skill ~/.codex/skills/Grok
```

### Grok itself

```bash
git clone https://github.com/gitguffaw/grok-claude-skill ~/.grok/skills/Grok
```

### Install once, share everywhere

```bash
git clone https://github.com/gitguffaw/grok-claude-skill ~/.claude/skills/Grok
ln -sfn ~/.claude/skills/Grok ~/.codex/skills/Grok
ln -sfn ~/.claude/skills/Grok ~/.grok/skills/Grok
```

### Upgrade

```bash
cd ~/.claude/skills/Grok && git pull
# or re-clone / rsync from this repo at tag v0.2.93
```

## Proven multi-turn spine

```bash
SID=$(grok -p "..." --output-format json --max-turns 8 --disable-web-search \
  | jq -r .sessionId)

grok -p "follow-up" --resume "$SID" --output-format json --max-turns 8
# or: -c for most recent session in this cwd
```

Create-only UUID (optional):

```bash
NEW=$(uuidgen | tr '[:upper:]' '[:lower:]')
grok -p "..." -s "$NEW" --output-format json
# later: --resume "$NEW"  — never reuse -s to continue
```

## How an agent uses it

1. Read `SKILL.md` — route mode + modifiers.
2. Open `references/QuickRef.md`, `references/LaunchPatterns.md`, and `Workflows/<Mode>.md` as needed.
3. Run `grok -p …` and inspect the result (diff, JSON, findings).

## Layout

```
SKILL.md                        # router: modes, modifiers, hard rules
CHANGELOG-P0.md                 # P0 re-verify notes
references/QuickRef.md          # flag surface
references/LaunchPatterns.md    # copy-paste launches
references/cli-surface.json     # machine snapshot of verified CLI surface
Workflows/                      # Analyze, Exec, Review, Parallel, Session
```

## License

[Apache License 2.0](LICENSE).
