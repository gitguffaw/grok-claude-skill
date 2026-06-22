# Grok skill

A portable **agent skill** that teaches an AI coding agent to drive the local **Grok CLI** (xAI) as an inner agent. It's written in the standard `SKILL.md` format, so it works in **Claude Code**, **Codex**, and **Grok** itself — any agent that loads `SKILL.md`-format skills and can run shell commands.

When the agent should delegate work to Grok instead of relying only on its own tools, this skill routes everything through headless `grok -p` with explicit control over model, effort, web search, tool scope, permissions, sessions, subagents/personas, best-of-N, and self-check.

## What it does

The skill treats Grok as a second agent and encodes:

- **Spine:** every delegation goes through headless `grok -p` (the one-shot run). `grok agent` (ACP / `stdio` / `serve`) is for IDE integration, not delegation, and the interactive TUI is left for the user.
- **Modes:** `Analyze` (read-only investigation), `Exec` (bounded change), `Review` (findings-only), `Parallel` (`--best-of-n` + subagents/personas), `Session` (multi-turn / machine-readable).
- **Modifiers:** model (`-m`), effort (`--effort low|medium|high|xhigh|max`), output format (`--output-format json|streaming-json`), permissions (`--always-approve` / `--permission-mode` / `--allow`/`--deny` / `--sandbox`), tool scope (`--tools` / `--disallowed-tools`), self-check (`--check`), plan mode, worktree isolation, and rules.
- **Guardrails:** the installed binary is treated as the source of truth over the bundled docs (verified drift is documented), models are resolved live with `grok models` (never pinned), and `grok inspect` is used to discover the real capability surface.

## Requirements

- The **`grok` CLI** installed and authenticated — `grok login`, or `XAI_API_KEY` for headless/CI. See <https://x.ai/cli>. Verified against `grok 0.2.39`.
- A host agent that loads `SKILL.md`-format skills: **Claude Code**, **Codex**, or **Grok**.

## Install

The skill is a single folder (`SKILL.md` + `references/` + `Workflows/`). Install it under the name `Grok` in your agent's skills directory. Use whichever agent(s) you run:

### Claude Code

```bash
# Personal — available in every project
git clone https://github.com/gitguffaw/grok-claude-skill ~/.claude/skills/Grok

# …or project-scoped — one repo only
git clone https://github.com/gitguffaw/grok-claude-skill <your-repo>/.claude/skills/Grok
```

Claude Code discovers skills at session start, so start a new session. Claude then invokes the skill automatically (via its Skill tool) when a task calls for delegating to Grok; you can also nudge it: *"use the Grok skill to …"*.

### Codex

Codex auto-discovers skills from `~/.codex/skills/` in the same `SKILL.md` format — drop the folder in, no CLI command needed:

```bash
git clone https://github.com/gitguffaw/grok-claude-skill ~/.codex/skills/Grok
```

Codex surfaces it alongside its other skills and invokes it when a task matches the description.

### Grok itself

Grok reads `~/.claude/skills/` and `~/.grok/skills/`, so the same folder works there too:

```bash
git clone https://github.com/gitguffaw/grok-claude-skill ~/.grok/skills/Grok
```

### Install once, share everywhere

Keep one copy and symlink it into the others — updates become a single `git pull`:

```bash
git clone https://github.com/gitguffaw/grok-claude-skill ~/.claude/skills/Grok
ln -s ~/.claude/skills/Grok ~/.codex/skills/Grok
ln -s ~/.claude/skills/Grok ~/.grok/skills/Grok
```

## How an agent uses it

The skill is plain instructions for driving the `grok` CLI — it doesn't rely on any one agent's internals. Whatever the host:

1. The agent reads `SKILL.md` to **route** the task — pick a mode (`Analyze` / `Exec` / `Review` / `Parallel` / `Session`) and any modifiers.
2. It consults `references/QuickRef.md` (flag-by-flag surface) and `references/LaunchPatterns.md` (copy-paste launch lines), and the matching `Workflows/<Mode>.md`.
3. It runs the chosen `grok -p …` command and inspects the result.

Because it's just markdown plus the `grok` binary, any agent that can read a file and run a shell command can follow it.

## Layout

```
SKILL.md                       # routing: modes, modifiers, launch rule, hard rules
references/QuickRef.md          # flag-by-flag CLI surface
references/LaunchPatterns.md    # copy-paste launch lines per mode/modifier
Workflows/                      # Exec, Analyze, Review, Parallel, Session
```

## License

[Apache License 2.0](LICENSE).
