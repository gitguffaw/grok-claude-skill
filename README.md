# Grok Skill

Teaches **Claude Code, Codex, Grok, or Google Antigravity** to drive the local **Grok CLI** through headless `grok -p`.

Clone this repo onto the host skill path. That directory is the skill (`SKILL.md`, `Workflows/`, `references/`).

## Prerequisites

1. Grok CLI installed: <https://x.ai/cli>
2. Authenticated: `grok login`, or `XAI_API_KEY` for headless/CI
3. `grok` on `PATH` where the host agent runs

```bash
grok --version
grok -p "Reply with: Grok CLI is ready"
```

## Install

```bash
git clone https://github.com/gitguffaw/Grok-Skill ~/.claude/skills/Grok
ln -sfn ~/.claude/skills/Grok ~/.codex/skills/Grok
ln -sfn ~/.claude/skills/Grok ~/.grok/skills/Grok
mkdir -p ~/.gemini/config/skills
ln -sfn ~/.claude/skills/Grok ~/.gemini/config/skills/Grok
```

If `~/.claude/skills/Grok` already exists, rename or remove it first.

Start a new host-agent session after installing.

| Host | Skill path |
|------|------------|
| Claude Code | `~/.claude/skills/Grok` |
| Codex | `~/.codex/skills/Grok` → Claude |
| Grok | `~/.grok/skills/Grok` → Claude |
| Antigravity global | `~/.gemini/config/skills/Grok` → Claude |
| Antigravity workspace | `<workspace>/.agents/skills/Grok` |

## Upgrade

```bash
cd ~/.claude/skills/Grok
git pull
```

Start a new host-agent session after upgrading.

## Skill layout

```text
SKILL.md                      # router: modes, launch, pointers
Workflows/                    # Analyze, Exec, Review, Parallel, Session
references/QuickRef.md        # verified CLI flag surface
references/LaunchPatterns.md
```

## Compatibility

| Field | Value |
|-------|-------|
| **Skill release** | `v1.0.8` (main) |
| **Verified Grok CLI** | `grok 1.0.5 (5115b46bc909) [stable]` |
| **Verified on** | 2026-08-21 |
| **Changelog** | [`CHANGELOG.md`](CHANGELOG.md) |

The installed `grok` binary is the source of truth. Probe `grok --help`, `grok models`, and `grok inspect` rather than assuming flags or model IDs.

## License

[Apache License 2.0](LICENSE)
