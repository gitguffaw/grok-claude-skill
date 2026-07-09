# CHANGELOG-P0 — Grok skill staging (vs live `~/.claude/skills/Grok/`)

**Stamp:** `grok 0.2.93 (f00f96316d4b) [stable]`, 2026-07-08  
**Basis:** `grok/verification/behavioral-report.md`, `doc-drift-ledger.md`, contracts (`behavioral-results.json`, `models-observed.json`, `cli-surface.json`)  
**Publish:** staging only under `grok/skill-staging/` — not installed to `~/.claude/skills/Grok/`.

## Blocking / high fixes

### Session spine
- **Removed all create-or-resume teaching for `-s/--session-id`.**
- **New semantics (proven):** `-s` is create-only, valid UUID, errors if exists or non-UUID.
- **Preferred multi-turn:** capture `sessionId` from `--output-format json` → `--resume "$SID"` or `-c`.
- **Added `--fork-session`** (with optional `-s` to name the forked UUID) as the only path combining `-s` with resume/continue.
- Removed “NOT shown in `grok --help`” claim — `-s` is documented on 0.2.93.
- `Workflows/Session.md` rewritten to proven recipes only; free-form named session examples gone.
- `SKILL.md` Router Session line, Hard Rules, Example 5 updated accordingly.
- `LaunchPatterns.md` Multi-Turn section rewritten; no create-or-resume language.

### Models
- Last Verified: **0.2.93 / 2026-07-08** (was 0.2.39 / 2026-06-10).
- Observed default: **`grok-4.5`** (point-in-time; was steady `grok-build`).
- Also seen: `grok-composer-2.5-fast`.
- Policy unchanged in spirit: resolve live via `grok models`; **no model IDs in Examples or LaunchPatterns** (`<MODEL>` only).

## Medium fixes (shipped with P0)

### Effort
- Primary flag: **`--reasoning-effort`**; **`--effort` is alias**.
- Levels documented: `none`, `minimal`, `low`, `medium`, `high`, `xhigh`, `max` — verify with help / rejection messages.
- Removed “headless-only” claim for effort; prefer verify-with-help.

### Headless-only set
- Dropped effort, permission-mode, and `-s` from headless-only list.
- Kept headless automation / help-marked: tools, disallowed-tools, max-turns, agents, check, best-of-n, output-format.

### New flags documented
- `--json-schema` (implies json; may add `structuredOutput`)
- `--fork-session`
- `--worktree-ref` / `--ref`

### Auto-approve wording
- Still launch with **`--always-approve`**.
- Note: historical short alias is a **hidden accepted CLI alias** (not help-listed), not “docs-only historical / TUI-only”.

### Output shapes
- json may include **`thought`**.
- `--json-schema` may add **`structuredOutput`**.
- streaming-json: `text`/`thought` have `data`; `end` has metadata.

## Other corrections

- Source hierarchy explicit: **binary > user-guide > root `~/.grok/*.md` > skill**.
- Web search still ON by default; `--disable-web-search` unchanged and correct.
- Tool IDs retained: `read_file`, `search_replace`, `grep`, `list_dir`, `run_terminal_cmd`, `web_search`, `web_fetch`, `todo_write`, `task`.
- Review patterns prefer embedding diff via command substitution / prompt file rather than pipe-only stdin.
- `GROK_HOME` not claimed as supported isolation (unproven on this binary); sessions keyed by cwd under `~/.grok/sessions/`.
- Compaction CLI flags kept but marked **may be hidden from `--help`**.
- `grok sessions` requires subcommand (`list|search|delete`).
- Optional machine extract: `references/cli-surface.json` (root flags; full surface remains in `grok/contracts/cli-surface.json`).

## Files in this staging package

```
skill-staging/
  SKILL.md
  CHANGELOG-P0.md
  Workflows/{Analyze,Exec,Review,Parallel,Session}.md
  references/{QuickRef.md,LaunchPatterns.md,cli-surface.json}
```

## Intentionally unchanged (still correct)

- Headless spine: `grok -p` / `--single`
- Prefer `--always-approve` for unattended runs
- Web search default + disable flag
- Mode set: Analyze / Exec / Review / Parallel / Session
- Progressive disclosure (router in SKILL.md, detail in refs/workflows)
- Do not drive interactive TUI from Claude
