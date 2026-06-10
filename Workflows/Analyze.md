# Analyze

## ENTER

- Repo-grounded unknowns dominate.
- You need facts, comparisons, or an explanation before acting.
- No file changes are wanted yet.

## AVOID

- A concrete change is the goal: use `Exec`.
- Only a critique of existing changes is wanted: use `Review`.

## OPTIONAL CONTROLS

- `Model`
- `Effort`
- `ToolScope` (read-only)
- `Machine`
- `WebSearch` (on by default)

## DEFAULT LAUNCH

Read-only investigation (no edits, no shell):

```bash
grok -p "<question>" --tools "read_file,grep,list_dir"
```

Allow web context (search is on by default, so no flag is needed):

```bash
grok -p "<question about current external facts and this repo>"
```

Model and effort for hard reasoning, keeping web context:

```bash
grok -p "<question>" -m <MODEL> --effort <high|xhigh|max> --tools "read_file,grep,list_dir,web_search,web_fetch"
```

Parseable answer:

```bash
grok -p "<question>" --tools "read_file,grep,list_dir" --output-format json | jq -r '.text'
```

## PROMPT SHAPE

```text
Question: <what must be answered>
Scope: <repo area, files, subsystem>
Output:
- facts (cite files and line ranges)
- options
- tradeoffs
- recommendation
Next decision: <what Claude will do after Grok answers>
```

## POST

- Read the answer.
- Decide: act via `Exec`, critique via `Review`, or deepen via `Parallel`.

## EXIT

- Uncertainty reduced.
- Or re-route to `Exec` / `Review` / `Parallel`.

## Examples

```bash
grok -p "Map how authentication flows from the HTTP layer to the session store in this repo. Cite files and line ranges. Do not edit." --tools "read_file,grep,list_dir"
```

## Current Facts

- A read-only allowlist (`read_file,grep,list_dir`) keeps the run from editing files or running commands.
- `--tools` disables default tool injection; add `web_search,web_fetch` back to the allowlist if you want web context.
- `--output-format json` returns `{text, stopReason, sessionId, requestId}`; capture `sessionId` to continue via `Session`.
