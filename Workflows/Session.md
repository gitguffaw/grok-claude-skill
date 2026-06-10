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

## DEFAULT LAUNCH

Named session (create-or-resume, recommended for scripting):

```bash
grok -p "<turn 1>" -s "<my-session-id>" --output-format json
grok -p "<turn 2>" -s "<my-session-id>" --output-format json
```

Capture the auto-assigned session id, then resume it:

```bash
SID=$(grok -p "<turn 1>" --output-format json | jq -r '.sessionId')
grok -p "<turn 2>" --resume "$SID" --output-format json | jq -r '.text'
```

Continue the most recent session for this directory:

```bash
grok -p "<next turn>" -c
```

Streaming events for real-time consumption:

```bash
grok -p "<task>" --output-format streaming-json
```

## POST

- Parse `.text` / `.sessionId` from the JSON object.
- Persist the session id if the thread continues.
- Inspect repo state if the turns made edits.

## EXIT

- Thread complete.
- Or re-route to `Exec` / `Review` / `Parallel` within the same session id.

## Examples

```bash
SID="critique-$(basename "$PWD")-pr"
grok -p "Review the changes in this PR. Findings only." -s "$SID" --disallowed-tools "search_replace,run_terminal_cmd" --output-format json | jq -r '.text'
grok -p "Now check only for security issues." -s "$SID" --output-format json | jq -r '.text'
```

## Current Facts

- `-s/--session-id` (create-or-resume) is headless-only and works, but is not shown in `grok --help`; `--resume <id>` (errors if missing) and `-c/--continue` (most recent for the cwd) are the help-verifiable equivalents.
- `--output-format json` returns `{text, stopReason, sessionId, requestId}` (the `.text` field is only on this object); `streaming-json` emits NDJSON `text` / `thought` events (each with a `data` field) and a terminal `end` event carrying `stopReason` / `sessionId` / `requestId`.
- `--restore-code` checks out the original session's commit when resuming; `--compaction-mode` / `--compaction-detail` tune context compaction on long threads.
- Headless starts a fresh session by default; pass `-s` (or `--resume`/`-c`) to maintain context across calls.
- `grok sessions list|search`, `grok export <id>`, and `grok import` manage the session store.
