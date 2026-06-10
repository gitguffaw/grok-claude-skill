# Review

## ENTER

- Need findings, not edits.
- Review target is uncommitted changes, a diff vs a base branch, or a specific commit.

## AVOID

- The same run should also implement fixes: use `Exec`.
- Parallel concern-specific review would materially improve coverage: use `Parallel`.

## OPTIONAL CONTROLS

- `Model`
- `Effort`
- `Machine`

## DEFAULT LAUNCH

Pipe the diff in as context and keep the run read-only:

```bash
git diff | grok -p "Review these changes for bugs, regressions, edge cases, and missing error handling. Findings only; do not edit." --disallowed-tools "search_replace,run_terminal_cmd"
```

Review staged changes:

```bash
git diff --staged | grok -p "Review the staged diff. Prioritized findings with file paths and concrete risk." --disallowed-tools "search_replace,run_terminal_cmd"
```

Let Grok read the tree itself, but block edits and shell:

```bash
grok -p "Review the uncommitted changes in this repo. Findings only." --tools "read_file,grep,list_dir"
```

Model, effort, and machine-readable:

```bash
git diff | grok -p "Review for security and correctness." -m <MODEL> --effort high --disallowed-tools "search_replace,run_terminal_cmd" --output-format json | jq -r '.text'
```

## OUTPUT CONTRACT

- prioritized findings
- file paths
- concrete risk
- suggested fix direction

## POST

- Fix manually, hand findings to `Exec`, or open a `Session` to review then fix in one thread.

## EXIT

- Findings accepted.
- Or re-route to `Exec` / `Parallel`.

## Examples

```bash
git diff main... | grok -p "Review this branch diff. Focus on regressions, edge cases, and maintainability. Findings only." --disallowed-tools "search_replace,run_terminal_cmd"
```

## Current Facts

- Grok has no dedicated `review` subcommand; review is a read-only headless prompt, optionally fed a diff over stdin.
- `--disallowed-tools "search_replace,run_terminal_cmd"` keeps tools available but removes editing and shell; `--tools "read_file,grep,list_dir"` is a stricter allowlist.
- Piped stdin is treated as additional context for the prompt.
