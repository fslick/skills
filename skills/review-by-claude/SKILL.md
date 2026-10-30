---
name: review-by-claude
description: Get an independent adversary review from Claude Code, run headless. Use when asked to have Claude review a plan, a diff or branch, or an executed plan, or to send disputes back to Claude for a re-review.
argument-hint: "'plan <path>', 'code', 'executed plan', or 're-review' with disputes"
---

# Review by Claude

This skill hands a review to Claude Code, which runs one of its own review skills (`request-plan-review` or `request-execution-review`) in headless mode. You are the caller, not the reviewer. Do not review the work yourself, and do not summarise or edit what Claude returns.

## Prerequisites

The review skills live at vault level, in `.claude/skills/` (a symlink to `.agents/skills/`), not in `~/.claude/skills/`. Claude finds them only when it starts in a directory that has them. Check from the directory you will run in, before spending a multi-minute call:

```bash
ls .claude/skills/request-plan-review/SKILL.md .claude/skills/request-execution-review/SKILL.md
```

If either is missing, stop and tell the user the review skills are not available in this directory.

The command needs network access and writes to `~/.claude`. If your harness runs commands in a sandbox (for example Codex's default `workspace-write`), request approval to run it outside the sandbox. If the sandbox blocks it, report that to the user; do not retry another way.

## Pick the mode

| Request | Claude prompt |
|---|---|
| Review a plan, spec, or design doc | `/request-plan-review <plan path>` |
| Review a diff, branch, or uncommitted code | `/request-execution-review code` |
| Check a finished implementation against its plan | `/request-execution-review executed plan` |
| Send disputes back after a review | Resume the session (see Re-review) |

If you were given a plan path, check that it exists and is non-empty before calling Claude. If it is missing, stop and tell the user.

## Run it

Run from the root of the repo under review, so Claude works out the git range itself. Pass the prompt on stdin with a quoted heredoc, so backticks, `$` and quotes reach Claude unchanged:

```bash
claude -p \
  --model sonnet \
  --effort medium \
  --setting-sources user,project \
  --settings '{"disableAllHooks":true}' \
  --append-system-prompt "Save the report copy to /tmp/." \
  --tools "Read,Grep,Glob,Bash,Write,Skill" \
  --allowedTools "Read,Grep,Glob,Skill,Edit(//tmp/review-*),Bash(git diff:*),Bash(git log:*),Bash(git show:*),Bash(git merge-base:*),Bash(git rev-parse:*),Bash(git status:*),Bash(ls:*)" \
  --disallowedTools "Bash(git * --output*)" \
  --output-format json <<'EOF'
<Claude prompt from the table>
EOF
```

What the flags do:

- `--model sonnet --effort medium` fixes the reviewer to the latest Sonnet at medium effort. Always pass both; never rely on Claude Code's default model, which another session may have changed.
- `--setting-sources user,project` loads the project's skills but drops `.claude/settings.local.json`, whose allow rules would otherwise add to the allowlist. A repo's `.claude/settings.json` does load; the vault has none.
- `--settings '{"disableAllHooks":true}'` turns off user hooks. A hook that rewrites commands (for example `git diff` to `rtk git diff`) would otherwise make the allowlisted git calls fail.
- `--tools` removes every other tool, including Edit.
- `Edit(//tmp/review-*)` limits writes to the report copy in `/tmp`. The review skills save to the OS temp dir, which is not `/tmp` on macOS, so `--append-system-prompt` pins the copy to `/tmp` while leaving the skill's argument clean. Any other write, and any command not on the list, is refused.
- `--disallowedTools` blocks git's `--output` flag, which would let `git diff` or `git log` write files.

Do not add `--dangerously-skip-permissions` or broaden the list.

A review can take several minutes. Set your command timeout to at least 600 seconds. If your harness cannot wait that long, run the command in the background with stdout redirected to a file, and read the file when it finishes.

The output is one JSON object. Use these fields:

- `result`: Claude's final message. Its last line is the path of the saved report copy. It may hold the full report or only that path, so read the report from the saved file.
- `session_id`: keep it, with the directory you ran from, for a re-review.
- `is_error`: if true, or the command fails, show the user the error and stop. Do not retry with wider permissions.

If `result` shows Claude did not run the review skill (for example, it says the skill is unknown), tell the user and stop.

## Re-review

After the author marks findings **Disputed** or **Uncertain**, resume the same Claude session so the reviewer keeps its earlier reasoning. Run it **from the same directory as the first call**; Claude stores sessions per directory, and a resume from elsewhere finds nothing.

```bash
claude -p --resume <session_id> <same flags as above> <<'EOF'
Re-Review

<the author's disputes, pasted in full>
EOF
```

Keep the `Re-Review` header; it tells Claude to start the next round. If the `session_id` is lost, run a fresh call instead with the same header, the previous report, and the disputes. Claude reads the round number from the report headings.

## Return the result

Give the user the saved report file's contents verbatim, followed by its path, then the `session_id` and run directory on their own lines so a later re-review can resume. Do not merge, renumber, or re-rank the findings. If `result` has no path line or the file is missing, return `result` verbatim instead and tell the user the saved copy was not found.
