# Grok Skill

Teaches **Claude Code, Codex, Grok, or Google Antigravity** to drive the local **Grok CLI** through headless `grok -p`.

The installable skill is the `Grok/` directory. Repo-root `README.md`, `LICENSE`, and `CHANGELOG.md` are human packaging, not skill files.

`~/.claude/skills/Grok` is **not** a git repo. Git commands run in the clone: `~/.claude/Grok-Skill`.

## Prerequisites

1. Grok CLI installed: <https://x.ai/cli>
2. Authenticated: `grok login`, or `XAI_API_KEY` for headless/CI
3. `grok` on `PATH` where the host agent runs

```bash
grok --version
grok -p "Reply with: Grok CLI is ready"
```

## Install

`git clone` creates the repo. Then symlink `Grok/` into each host:

```bash
git clone --branch v1.0.8 https://github.com/gitguffaw/Grok-Skill ~/.claude/Grok-Skill
ln -sfn ~/.claude/Grok-Skill/Grok ~/.claude/skills/Grok
ln -sfn ~/.claude/skills/Grok ~/.codex/skills/Grok
ln -sfn ~/.claude/skills/Grok ~/.grok/skills/Grok
mkdir -p ~/.gemini/config/skills
ln -sfn ~/.claude/skills/Grok ~/.gemini/config/skills/Grok
```

If `~/.claude/skills/Grok` already exists and is **not** that symlink, rename or remove it first, then run the `ln` lines.

Start a new host-agent session after installing.

| Host | Skill path |
|------|------------|
| Claude Code | `~/.claude/skills/Grok` → `~/.claude/Grok-Skill/Grok` |
| Codex | `~/.codex/skills/Grok` → Claude skill path |
| Grok | `~/.grok/skills/Grok` → Claude skill path |
| Antigravity global | `~/.gemini/config/skills/Grok` → Claude skill path |
| Antigravity workspace | `<workspace>/.agents/skills/Grok` |

## Upgrade

Only if `~/.claude/Grok-Skill` is already a clone of this repo:

```bash
cd ~/.claude/Grok-Skill
git fetch --tags
git checkout v1.0.8
```

Do not `cd ~/.claude/skills/Grok` and run git there.

If you do not have `~/.claude/Grok-Skill/.git`, skip upgrade and run **Install**.

Start a new host-agent session after upgrading.

## Skill layout

```text
Grok/SKILL.md                 # router: modes, launch, pointers
Grok/Workflows/               # Analyze, Exec, Review, Parallel, Session
Grok/references/QuickRef.md   # verified CLI flag surface
Grok/references/LaunchPatterns.md
```

## Compatibility

| Field | Value |
|-------|-------|
| **Skill release** | `v1.0.8` |
| **Verified Grok CLI** | `grok 1.0.5 (5115b46bc909) [stable]` |
| **Verified on** | 2026-08-21 |
| **Changelog** | [`CHANGELOG.md`](CHANGELOG.md) |

The installed `grok` binary is the source of truth. Probe `grok --help`, `grok models`, and `grok inspect` rather than assuming flags or model IDs.

## License

[Apache License 2.0](LICENSE)
