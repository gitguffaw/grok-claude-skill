# Grok Skill

An agent skill that teaches **Claude Code, Codex, Grok, or Google Antigravity** how to use the local **Grok CLI** as a second coding and analysis agent.

This repository does **not** install Grok itself. The skill launches the `grok` executable already installed on your machine, primarily through headless `grok -p` commands.

## Prerequisites

Before installing this skill, you need:

1. **Grok CLI installed.** Follow the official instructions at <https://x.ai/cli>.
2. **Grok authenticated.** Run `grok login`, or provide `XAI_API_KEY` in headless or CI environments.
3. **A supported host agent:** Claude Code, Codex, Grok, or Google Antigravity.
4. **`grok` available on `PATH`** in the environment where your host agent runs.

Verify the prerequisite before continuing:

```bash
grok --version
grok -p "Reply with: Grok CLI is ready"
```

If either command fails, install or authenticate Grok before installing the skill.

## Quick start

### 1. Install the skill

Install it under the name **`Grok`** (TitleCase). For one shared installation across all supported hosts:

```bash
# Canonical installation
git clone --branch v0.2.94 https://github.com/gitguffaw/Grok-Skill ~/.claude/skills/Grok

# Let the other hosts use the same installation
ln -sfn ~/.claude/skills/Grok ~/.codex/skills/Grok
ln -sfn ~/.claude/skills/Grok ~/.grok/skills/Grok
mkdir -p ~/.gemini/config/skills
ln -sfn ~/.claude/skills/Grok ~/.gemini/config/skills/Grok
```

Start a new host-agent session after installation so it discovers the skill.

### 2. Ask your host to use Grok

Describe the task normally and explicitly ask for Grok when you want delegation. For example:

```text
Use Grok to analyze this repository and identify the likely cause of the failing tests.
```

```text
Have Grok review the current diff for correctness and security issues. Do not edit files.
```

```text
Use Grok to implement this change, run the relevant tests, and inspect the resulting diff.
```

The host loads this skill, chooses the appropriate workflow, and launches your local Grok CLI. You do not need to construct the full `grok -p` command yourself.

## What the skill adds

- **Headless delegation:** uses `grok -p` so another agent can invoke Grok and capture its result.
- **Task modes:** `Analyze`, `Exec`, `Review`, `Parallel`, and `Session`.
- **Explicit controls:** model, reasoning effort, web search, tool scope, permissions, sessions, subagents/personas, best-of-N, worktrees, and self-check.
- **Safety guidance:** separates read-only review from editing workflows and prefers tight tool allowlists where appropriate.
- **Session handling:** correctly continues conversations with `--resume` or `-c` instead of reusing the create-only `-s` flag.

## Install for one host only

### Claude Code

```bash
git clone --branch v0.2.94 https://github.com/gitguffaw/Grok-Skill ~/.claude/skills/Grok
```

For a project-scoped installation, clone into `<repo>/.claude/skills/Grok` instead.

### Codex

```bash
git clone --branch v0.2.94 https://github.com/gitguffaw/Grok-Skill ~/.codex/skills/Grok
```

### Grok

```bash
git clone --branch v0.2.94 https://github.com/gitguffaw/Grok-Skill ~/.grok/skills/Grok
```

### Google Antigravity

```bash
# Global installation
mkdir -p ~/.gemini/config/skills
git clone --branch v0.2.94 https://github.com/gitguffaw/Grok-Skill ~/.gemini/config/skills/Grok

# For a workspace-only installation, clone into:
# <workspace>/.agents/skills/Grok
```

| Host | Skill path |
|------|------------|
| Claude Code | `~/.claude/skills/Grok` |
| Codex | `~/.codex/skills/Grok` |
| Grok | `~/.grok/skills/Grok` |
| Antigravity global | `~/.gemini/config/skills/Grok` |
| Antigravity workspace | `<workspace>/.agents/skills/Grok` |

## Upgrade

For the shared installation shown above:

```bash
cd ~/.claude/skills/Grok
git fetch --tags
git checkout v0.2.94
```

Symlinked hosts use the updated files automatically. Start a new host-agent session after upgrading.

## Direct CLI example

The skill handles this for you, but its multi-turn command pattern looks like this:

```bash
SID=$(grok -p "Analyze this repository" \
  --output-format json \
  --max-turns 8 \
  --disable-web-search | jq -r .sessionId)

grok -p "Now summarize the highest-risk issue" \
  --resume "$SID" \
  --output-format json \
  --max-turns 8
```

`-s/--session-id` creates a new conversation with a specific UUID. Use `--resume` or `-c` to continue an existing conversation.

## Compatibility and verification

| Field | Value |
|-------|-------|
| **Skill release** | `v0.2.94` |
| **Verified Grok CLI** | `grok 0.2.93 (f00f96316d4b) [stable]` |
| **Verified on** | 2026-07-08 (CLI contracts and packaging) |
| **Changelog** | [`CHANGELOG.md`](CHANGELOG.md) |

The installed binary is the source of truth. Grok models and CLI capabilities can change, so the skill probes commands such as `grok --help`, `grok models`, and `grok inspect` instead of assuming stale model IDs or flags remain available.

## Repository layout

```text
SKILL.md                        # routing policy, modes, modifiers, and hard rules
CHANGELOG.md                    # release history
references/QuickRef.md          # verified CLI flag surface
references/LaunchPatterns.md    # copy-paste launch patterns
references/cli-surface.json     # captured CLI surface
Workflows/                      # Analyze, Exec, Review, Parallel, and Session
```

## License

[Apache License 2.0](LICENSE)
