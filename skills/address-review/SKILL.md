---
name: address-review
description: Verify an adversary review's findings against your own plan or code. Use when handed one or more review reports to address, or asked to respond to reviewer findings.
argument-hint: "The review report(s) to address - pasted text or a file path"
disable-model-invocation: false
---

# Address Review

You own this work; an **adversary reviewer** (often a different, stronger model) just critiqued it. The reviewer has no stake in your choices and no context you didn't give it, so a finding is a claim to **verify**, never an order to obey. Your job: accept what's right, implement it, and push back on the rest with technical reasoning.

## Git is read-only

You may edit files in the working tree. You may **not** commit, stage, branch, checkout, stash, reset, or run any mutating git operation. Leave the tree dirty for the human.

## Validate the input first

Check every given file path exists and is non-empty before assessing anything. If any is missing or empty, stop and report it to the user. Do not proceed, and do not treat it as a clean review.

## Pick the mode

- **Plan mode**: you're addressing a review of a plan document; accepted findings mean editing the plan `.md`.
- **Code mode**: you're addressing a review of code; accepted findings mean editing source.

Detect from the report and target. If unclear, ask.

## Merge multiple reports

You may be handed several reports (different models reviewing the same work). They don't know about each other. **De-duplicate** overlapping findings into one merged list before assessing. Never implement or dispute the same thing twice.

## Track the round

Read a round only from a **headline** in the report(s) you're addressing (`## … – Round N`), never from prose inside a report body. Your output is Round N+1. No headline round → Round 1.

Rounds count within one review cycle, so a plan cycle and an execution cycle each run their own 1, 2, 3… Name the cycle in your heading (`Addressed Plan Review` in plan mode, `Addressed Execution Review` in code mode) so two cycles in the same window stay distinguishable.

## Assess each finding - don't perform

For every finding:

1. **Verify it against reality**: read the code/plan, check whether the claim actually holds *for this codebase*. Don't accept on the reviewer's authority.
2. **YAGNI**: if the reviewer wants a feature, grep for whether it's used. Unused → dispute it, don't build it.
3. Decide: **accept**, **dispute**, or **uncertain**.

No performative agreement. Never "you're absolutely right", "great catch", or any thanks. State verdicts and reasoning; the edits speak for themselves.

When the work was reviewed by a stronger model, the pull is to cave on everything. Resist it: a finding you can confidently refute gets disputed; a finding you genuinely **cannot verify either way** goes to **Uncertain**. Never silently accept it, never manufacture a dispute to look rigorous.

## Implement the accepted findings

Edit the plan or source for every accepted finding. One at a time. Don't commit.

## Output: three buckets

The template shows plan mode at Round 2; write your actual heading and round number. Both headings carry the same number.

```markdown
## Addressed Plan Review – Round 2

### Accepted & implemented
- **[Title]** - [what you changed, file:line]

## Re-Review – Round 2

### Disputed
- **[Title]** - [why the finding is wrong, with technical reasoning]

### Uncertain
- **[Title]** - [what you couldn't verify and what would settle it]
```

**Accepted & implemented** is informational: done, not for the next round.

The `## Re-Review` block (Disputed + Uncertain) is the artifact you paste back into the reviewer's window for round N+1. The `Re-Review` header is the signal that tells `/request-plan-review` or `/request-execution-review` to defend or concede each, not re-review from scratch. **Uncertain** stays in this block: surface those for the human and the reviewer rather than resolving them silently.

After the output above:
1. Call a file-write tool and save a copy of the full output to the OS temp dir as `disputes-<slug>-<UTC-timestamp>.md`.
2. Print that path on one trailing line, only once the tool call has actually succeeded. If you haven't called the write tool yet, call it now before printing anything.

Group findings under Critical/Important/Minor only where it adds signal. Don't force empty severity headings.
