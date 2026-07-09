# Grok skill

A portable **agent skill** that teaches an AI coding agent to drive the local **Grok CLI** (xAI) as an inner agent. Written in the standard `SKILL.md` format for **Claude Code**, **Codex**, **Grok**, and **Google Antigravity** (AGY / AGY IDE / AGY CLI).

When the host should delegate to Grok instead of using only its own tools, this skill routes through headless `grok -p` with explicit control over model, effort, web search, tool scope, permissions, sessions, subagents/personas, best-of-N, and self-check.

## Release stamp

| Field | Value |
|-------|--------|
| **Skill release** | `v0.2.94` |
| **Verified binary** | `grok 0.2.93 (f00f96316d4b) [stable]` (unchanged from P0) |
| **Verified on** | 2026-07-08 (CLI contracts); packaging 2026-07-08 |
| **Changelog** | [`CHANGELOG.md`](CHANGELOG.md) |

**CLI contract highlights (since 0.2.39-era skill):**

- Multi-turn: capture `sessionId` from `--output-format json`, then **`--resume` / `-c`**. Do **not** reuse `-s` to continue.
- `-s/--session-id` is **create-only** (valid UUID, errors if already in use).
- Review / findings-only: prefer `--tools "read_file,grep,list_dir"`. Denylist form must include **`write`**.
- Models: resolve live with `grok models` (observed default at verify time: `grok-4.5`).

## What it does

- **Spine:** headless `grok -p` for delegation. `grok agent` is ACP/IDE; interactive TUI is user-facing.
- **Modes:** `Analyze`, `Exec`, `Review`, `Parallel`, `Session`.
- **Modifiers:** model, effort (`--reasoning-effort` / `--effort`), output format, `--json-schema`, permissions, tool scope, self-check, worktree, rules, fork-session.
- **Guardrails:** installed binary beats bundled docs; models never pinned in recipes; `grok inspect` for discovered capabilities.

## Requirements

- **`grok` CLI** installed and authenticated (`grok login`, or `XAI_API_KEY` for headless/CI). See <https://x.ai/cli>.
- Host that loads `SKILL.md` skills: **Claude Code**, **Codex**, **Grok**, or **Antigravity**.
- Ensure `grok` is on `PATH` inside the host agent’s environment.
- CLI contracts last verified against **`grok 0.2.93`**. After upgrading Grok, re-check `grok --version` against `SKILL.md`.

## Install

Install under the name **`Grok`** (TitleCase).

### Recommended: one clone, symlink everywhere

```bash
# Canonical install (Claude Code)
git clone --branch v0.2.94 https://github.com/gitguffaw/grok-claude-skill ~/.claude/skills/Grok

# Host discovery paths (same content, one source of truth)
ln -sfn ~/.claude/skills/Grok ~/.codex/skills/Grok
ln -sfn ~/.claude/skills/Grok ~/.grok/skills/Grok
mkdir -p ~/.gemini/config/skills
ln -sfn ~/.claude/skills/Grok ~/.gemini/config/skills/Grok   # Antigravity global
```

| Host | Path |
|------|------|
| Claude Code | `~/.claude/skills/Grok` |
| Codex | `~/.codex/skills/Grok` |
| Grok | `~/.grok/skills/Grok` |
| **Antigravity** (AGY / IDE / CLI global) | `~/.gemini/config/skills/Grok` |
| Antigravity workspace-only | `<workspace>/.agents/skills/Grok` |

`~/.gemini/config/skills/` is the global path recognized across Antigravity products. Workspace skills use `.agents/skills/` under the project root.

Start a **new session** after install so the host reloads skills.

### Claude Code only

```bash
git clone --branch v0.2.94 https://github.com/gitguffaw/grok-claude-skill ~/.claude/skills/Grok
# or project-scoped:
# git clone --branch v0.2.94 https://github.com/gitguffaw/grok-claude-skill <repo>/.claude/skills/Grok
```

### Codex only

```bash
git clone --branch v0.2.94 https://github.com/gitguffaw/grok-claude-skill ~/.codex/skills/Grok
```

### Grok only

```bash
git clone --branch v0.2.94 https://github.com/gitguffaw/grok-claude-skill ~/.grok/skills/Grok
```

### Antigravity only

```bash
# Global (preferred for all AGY surfaces)
mkdir -p ~/.gemini/config/skills
git clone --branch v0.2.94 https://github.com/gitguffaw/grok-claude-skill ~/.gemini/config/skills/Grok

# Workspace-only
# mkdir -p .agents/skills
# git clone --branch v0.2.94 https://github.com/gitguffaw/grok-claude-skill .agents/skills/Grok
```

### Upgrade

```bash
cd ~/.claude/skills/Grok && git fetch --tags && git checkout v0.2.94
# symlinked hosts pick up the same commit automatically
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
CHANGELOG.md                    # release history
references/QuickRef.md          # flag surface
references/LaunchPatterns.md    # copy-paste launches
references/cli-surface.json     # machine snapshot of verified CLI surface
Workflows/                      # Analyze, Exec, Review, Parallel, Session
```

## License

[Apache License 2.0](LICENSE).
