# Grok skill for Claude Code

A [Claude Code](https://docs.claude.com/en/docs/claude-code) **skill** that drives the local **Grok CLI** (xAI) as Claude's inner coding and analysis agent. When Claude should delegate work to Grok instead of relying only on its own tools, this skill teaches it to route everything through headless `grok -p` with explicit control over model, effort, web search, tool scope, permissions, sessions, subagents/personas, best-of-N, and self-check.

## What it does

Outer Claude treats Grok as a second agent. The skill encodes:

- **Spine:** every delegation goes through headless `grok -p` (the one-shot agent run). `grok agent` (ACP / `stdio` / `serve`) is for IDE integration, not delegation, and the interactive TUI is left for the user — Claude does not drive it.
- **Modes:** `Analyze` (read-only investigation), `Exec` (bounded change), `Review` (findings-only), `Parallel` (`--best-of-n` + subagents/personas), `Session` (multi-turn / machine-readable).
- **Modifiers:** model (`-m`), effort (`--effort low|medium|high|xhigh|max`), output format (`--output-format json|streaming-json`), permissions (`--always-approve` / `--permission-mode` / `--allow`/`--deny` / `--sandbox`), tool scope (`--tools` / `--disallowed-tools`), self-check (`--check`), plan mode, worktree isolation, and rules.
- **Guardrails:** the installed binary is treated as the source of truth over the bundled docs (verified drift is documented), models are resolved live with `grok models` (never pinned), and `grok inspect` is used to discover the real capability surface.

## Requirements

- The `grok` CLI installed and authenticated (`grok login`, or `XAI_API_KEY` for headless/CI). See <https://x.ai/cli>.
- Claude Code — the skill is loaded and invoked by Claude, not run directly.

Verified against `grok 0.2.39`.

## Install

Clone into your Claude skills directory as `Grok`:

```bash
git clone https://github.com/gitguffaw/grok-claude-skill ~/.claude/skills/Grok
```

Claude Code discovers it on the next launch. (Grok itself also reads `~/.claude/skills`, so the same folder doubles as a Grok skill.)

## Layout

```
SKILL.md                      # routing: modes, modifiers, launch rule, hard rules
references/QuickRef.md         # flag-by-flag CLI surface
references/LaunchPatterns.md   # copy-paste launch lines per mode/modifier
Workflows/                     # Exec, Analyze, Review, Parallel, Session
```

## License

[Apache License 2.0](LICENSE).
