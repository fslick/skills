---
name: review-by-codex
description: Delegate an independent review to Codex CLI for a plan, code, an executed plan, or a re-review of disputes.
argument-hint: "'plan <path>', 'code', 'executed plan', or 're-review' with disputes"
---

# Review by Codex

You are the caller, not the reviewer. Launch a fresh Codex session and relay its report verbatim. The reviewer reads the shared review skill; do not copy its criteria or the author's conversation into the prompt.

## Prepare

1. Find the absolute, physical directory containing this `SKILL.md`. Its sibling `../request-plan-review/SKILL.md` handles `plan`; `../request-execution-review/SKILL.md` handles `code` and `executed plan`. Resolve the selected path and require a readable, nonempty file. Require `codex` on PATH, and check the installed `codex exec --help` and `codex exec resume --help` support the flags below. Stop if any prerequisite fails.
2. Identify the exact project directory and use its absolute path as the working directory. For `plan <path>`, require a readable, nonempty plan; pass its absolute path. For `executed plan`, validate any supplied plan path in the same way; otherwise ask the reviewer to locate it and say what it found. For `code` and `executed plan`, require a real Git repository before launch, using `git rev-parse --show-toplevel` in the project directory; never run Git in a directory known not to be a repository, such as Slick Vault. Do not initialise a repository.
3. Create a fresh private directory under `/tmp/` with `mktemp -d`. Keep the prompt, JSONL events, stderr and report there, in separate files. Write the prompt as literal data, for example with a quoted heredoc; never interpolate review text into shell code. Tell the child: read the selected skill by absolute path, perform the selected review in the project, and follow its report format and criteria. Override only its final file-writing and path-printing instructions: return the complete report as the final message, without writing a report or printing a path. Supply only the mode, plan path if any, and material needed for the review.
4. Before each launch, inspect the installed CLI's effective server configuration from the project directory: `codex mcp list --json --disable hooks --disable plugins --disable apps -c 'notify=[]'`. Build one `-c` TOML map for `mcp_servers` containing every returned name and its transport identity with `enabled=false`: preserve the configured command for a stdio server or URL for an HTTP server. Encode names and values as TOML strings, including names containing dots; construct the argument as data, not shell source. Config layers merge, so changing a transport type can leave incompatible inherited fields. Keep credential-bearing configuration private. Run the same list command with all those overrides and require every returned server to be disabled. Also inspect `codex app-server --print-startup-config --disable hooks --disable plugins --disable apps` and stop if managed requirements or hooks prevent you from verifying the boundary. Retain the exact override arguments for launch and resume. If inspection fails, an integration remains enabled, or the CLI does not support these controls, stop rather than run a review. Recheck immediately before launch, including on resume.

The reviewer uses Codex's read-only filesystem sandbox and never receives approval for unsupported actions. MCP servers, apps, plugins, hooks, and notification commands are disabled as above; this is not a shell-command allowlist. Read-only filesystem access is not itself a remote-service restriction. Codex still needs service connectivity and can write its own authentication and session state. If an outer sandbox blocks these, request the permission required to run the concrete command while retaining the child sandbox. If denied, report the failure; do not bypass the child sandbox or retry with broader access.

## Fresh review

Use the following invocation, replacing the placeholders with individual arguments and redirecting the literal prompt from its file. `<server overrides>` is the complete list verified above, not a single literal argument. `--skip-git-repo-check` permits non-Git plan review; it does not make code review outside a repository valid.

```bash
codex exec --ignore-rules --disable hooks --disable plugins --disable apps \
  -c 'notify=[]' <server overrides> -c 'approval_policy="never"' \
  -c 'model_reasoning_effort="high"' -s read-only -m gpt-6.1-terra \
  -C "$project" --skip-git-repo-check --json -o "$run/report.md" - \
  < "$run/prompt.txt" > "$run/events.jsonl" 2> "$run/stderr.txt"
```

Pin the given model and effort. If unavailable, report the error; do not silently substitute. Allow at least ten minutes where supported; otherwise use the host's background-process facility and poll its handle. A timeout or interruption is incomplete: retain the artifacts and do not automatically launch a duplicate.

## Re-review

Use the exact recorded thread ID and original absolute project directory. Require the full disputes, and check any supplied report or dispute file exists and is nonempty. Put `Re-Review` followed by the full disputes into the prompt as literal data. The original reviewer retains its earlier reasoning and determines the next round using the shared skill.

Run from the original project directory. Resume has different options, so set sandbox mode with configuration and do not pass `-C` or `-s` after `resume`:

```bash
codex exec resume --ignore-rules --disable hooks --disable plugins --disable apps \
  -c 'notify=[]' <server overrides> -c 'approval_policy="never"' \
  -c 'sandbox_mode="read-only"' -c 'model_reasoning_effort="high"' \
  -m gpt-6.1-terra --skip-git-repo-check --json -o "$run/report.md" \
  "$thread_id" - < "$run/prompt.txt" > "$run/events.jsonl" 2> "$run/stderr.txt"
```

Use a fresh `/tmp/` run directory for this round and preserve the previous artifacts. If the session is unavailable, explicitly label a fresh review as recovery and include the complete previous report and disputes under `Re-Review`; the shared skill derives the next round from report headlines. Never use `--last`.

## Validate and return

Wait for the process to exit. Success requires exit status zero, valid complete JSONL, a `thread.started.thread_id`, a `turn.completed` event, no `turn.failed` or error events, a nonempty report, and evidence the child could read and follow the selected review skill. For resume, require the returned thread ID to equal the requested ID. A later failure invalidates even an already written report. Missing artifacts, interrupted or truncated events, an unreadable skill, or any failed condition are incomplete results, never a clean review; return available diagnostics without secrets.

On success, copy the saved Markdown report byte-for-byte into the response, preserving every Unicode character, punctuation mark and newline; do not retype, paraphrase or normalize quotes. Before sending, compare the report portion of the prepared response with the saved file byte-for-byte and correct any difference. Follow it with its actual path, thread ID, absolute project directory, model, effort and effective safety options. Do not merge, edit, renumber, or summarise findings. Keep the logs for diagnosis and preserve the exact identity and options for resume.
