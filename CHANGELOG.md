# Changelog

## v1.0.6 — 2026-08-21

Skill package `v1.0.6`. Re-verify against `grok 1.0.5 (5115b46bc909) [stable]`.

### Breaking vs v0.2.94 skill recipes

- **`--check` / `--self-verify` gone.** Parser rejects them. Self-verification belongs in the prompt Validation section.
- **`--best-of-n` gone.** Parser rejects it. `Parallel` is subagent orchestration; independent attempts are N host-side `grok -p` runs.
- **`grok import` gone.** Session movement is `grok export` plus `grok sessions`.
- **Headless `-w` does not create a worktree.** Isolate with `--cwd` into an existing worktree, or inner subagent worktree isolation.
- **Live effort accept list is `low|medium|high|xhigh`.** User-guide extra tiers (`none`, `minimal`, `max`) currently reject.

### Added / updated

- Output format `streaming-messages-json` and `--include-partial-messages`.
- `--resume` matches session ID **or** title for the current directory (scripts should still pass IDs).
- `--restore-code` restores a session snapshot on resume; remote sessions require `--worktree`.
- JSON output may include spend fields (`usage`, `num_turns`, `modelUsage`, cost).
- Secondary commands: `grok doctor`, `grok du` (`disk-usage`), `grok wrap`.
- User-guide documents `GROK_HOME` and `GROK_MEMORY=1|0` (memory still experimental, off by default).
- Observed default model: `grok-4.6` (point-in-time).

### Unchanged

- Headless spine `grok -p`; create-only `-s`; multi-turn via `--resume` / `-c`.
- Prefer `--always-approve` (the short parser alias remains hidden from `grok --help`).
- Review allowlist `read_file,grep,list_dir`; denylist must include `write`.

## v0.2.94 — 2026-07-08

### Added

- **Google Antigravity** install paths:
  - Global: `~/.gemini/config/skills/Grok` (AGY / AGY IDE / AGY CLI)
  - Workspace: `<workspace>/.agents/skills/Grok`
- Multi-host **symlink** install as the recommended default (one clone under Claude Code, link Codex / Grok / Antigravity).

### Unchanged

- CLI contracts still those verified against **`grok 0.2.93`** (P0). No headless semantic changes in this packaging release.

## v0.2.93 — 2026-07-08

P0 re-verify against `grok 0.2.93`: create-only session UUID, resume multi-turn spine, write-safe review allowlist, surface snapshot, invariant-checked packaging. See `CHANGELOG-P0.md` for the full P0 delta.
