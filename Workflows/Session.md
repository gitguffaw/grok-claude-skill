# Session

## ENTER

- Work spans multiple `grok -p` calls and must keep context, OR
- Claude needs machine-readable output to parse or chain.

## AVOID

- A single self-contained run is enough: use `Exec` / `Analyze` / `Review`.

## OPTIONAL CONTROLS

- `Model`
- `Effort`
- `Machine`
- `Permissions`
- `JsonSchema`

## DEFAULT LAUNCH

Preferred multi-turn spine (capture auto session id, then resume):

```bash
SID=$(grok -p "<turn 1>" --output-format json --always-approve \
  | jq -r '.sessionId')
grok -p "<turn 2>" --resume "$SID" --output-format json --always-approve \
  | jq -r '.text'
```

Continue the most recent session for this directory:

```bash
grok -p "<next turn>" -c --output-format json
```

Optional client-chosen UUID for a **new** session only (must be a valid UUID; must not already exist):

```bash
UUID=$(uuidgen | tr '[:upper:]' '[:lower:]')
grok -p "<turn 1>" -s "$UUID" --output-format json
# later: always resume — do NOT reuse -s
grok -p "<turn 2>" --resume "$UUID" --output-format json
```

Fork into a new session while keeping history (optional `-s` names the fork UUID):

```bash
FORK=$(uuidgen | tr '[:upper:]' '[:lower:]')
grok -p "<branch of work>" --resume "$SID" --fork-session -s "$FORK" --output-format json
# or auto-assign the forked id:
grok -p "<branch of work>" --resume "$SID" --fork-session --output-format json
```

Streaming events for real-time consumption:

```bash
grok -p "<task>" --output-format streaming-json
```

Structured output (implies JSON; may include `structuredOutput`):

```bash
grok -p "<task>" --json-schema '{"type":"object","properties":{"ok":{"type":"boolean"}},"required":["ok"]}'
```

## POST

- Parse `.text` / `.sessionId` from the JSON object (`.thought` may also be present).
- With `--json-schema`, prefer `.structuredOutput` when present.
- Persist the session id if the thread continues.
- Inspect repo state if the turns made edits.

## EXIT

- Thread complete.
- Or re-route to `Exec` / `Review` / `Parallel` within the same session via `--resume` / `-c`.

## Examples

```bash
SID=$(grok -p "Review the changes in this PR. Findings only. Do not edit." \
  --tools "read_file,grep,list_dir" \
  --output-format json | jq -r '.sessionId')
grok -p "Now check only for security issues. Findings only." \
  --resume "$SID" --tools "read_file,grep,list_dir" --output-format json | jq -r '.text'
```

## Current Facts

- Multi-turn that works: capture `sessionId` from JSON → `--resume $SID` or `-c` for most recent in cwd.
- `-s/--session-id` is **create-only**: valid UUID required; reusing an existing UUID errors (`Session ID … is already in use`); non-UUID IDs error. Documented in `grok --help`. Never teach create-or-resume with `-s`.
- With `--resume` / `--continue`, `-s` is only valid together with `--fork-session` (names the forked session).
- `--fork-session` creates a new session ID on resume while retaining history.
- `--output-format json` returns `{text, stopReason, sessionId, requestId}` and may include `thought`; `--json-schema` implies json and may add `structuredOutput`.
- `streaming-json`: NDJSON `text`/`thought` events carry `data`; terminal `end` carries `stopReason` / `sessionId` / `requestId`.
- `--restore-code` checks out the original session's commit when resuming.
- Compaction CLI flags exist but may be hidden from `grok --help`: `--compaction-mode` / `--compaction-detail` (env `GROK_COMPACTION_MODE` / `GROK_COMPACTION_DETAIL`) — verify acceptance if needed.
- Headless starts a fresh session by default; pass `--resume`/`-c` (or create-only `-s` then later resume) to maintain context across calls.
- Sessions are keyed by cwd under `~/.grok/sessions/`. Isolate scripted work with a unique `--cwd` / process cwd when needed.
- `grok sessions list|search|delete`, `grok export <id>`, and `grok import` manage the session store.
