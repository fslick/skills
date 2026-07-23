---
name: address-review
description: Assess external review reports against reality, implement what you agree with, and emit a stamped re-review block.
argument-hint: "The review report(s) to address - pasted text or a file path"
disable-model-invocation: false
---

# Address Review

You own this work; an **adversary reviewer** (often a different, stronger model) just critiqued it. The reviewer has no stake in your choices and no context you didn't give it - so a finding is a claim to **verify**, never an order to obey. Your job: accept what's right, implement it, and push back on the rest with technical reasoning.

## Git is read-only

You may edit files in the working tree. You may **not** commit, stage, branch, checkout, stash, reset, or run any mutating git operation. Leave the tree dirty for the human.

## Pick the mode

- **Plan mode** - you're addressing a review of a plan document; accepted findings mean editing the plan `.md`.
- **Code mode** - you're addressing a review of code; accepted findings mean editing source.

Detect from the report and target. If unclear, ask.

## Merge multiple reports

You may be handed several reports (different models reviewing the same work). They don't know about each other. **De-duplicate** overlapping findings into one merged list before assessing - never implement or dispute the same thing twice.

## Assess each finding - don't perform

For every finding:

1. **Verify it against reality** - read the code/plan, check whether the claim actually holds *for this codebase*. Don't accept on the reviewer's authority.
2. **YAGNI** - if the reviewer wants a feature, grep for whether it's used. Unused → dispute it, don't build it.
3. Decide: **accept**, **dispute**, or **uncertain**.

No performative agreement - never "you're absolutely right", "great catch", or any thanks. State verdicts and reasoning; the edits speak for themselves.

When the work was reviewed by a stronger model, the pull is to cave on everything. Resist it: a finding you can confidently refute gets disputed; a finding you genuinely **cannot verify either way** goes to **Uncertain** - never silently accept it, never manufacture a dispute to look rigorous.

## Implement the accepted findings

Edit the plan or source for every accepted finding. One at a time. Don't commit.

## Output: three buckets

```markdown
## Addressed Review

### Accepted & implemented
- **[Title]** - [what you changed, file:line]

## Re-Review

### Disputed
- **[Title]** - [why the finding is wrong, with technical reasoning]

### Uncertain
- **[Title]** - [what you couldn't verify and what would settle it]
```

**Accepted & implemented** is informational - done, not for the next round.

The `## Re-Review` block (Disputed + Uncertain) is the artifact you paste back into the reviewer's window for round N+1 - the `Re-Review` header is the signal that tells `/request-plan-review` or `/request-execution-review` to defend or concede each, not re-review from scratch. **Uncertain** stays in this block: surface those for the human and the reviewer rather than resolving them silently.

Save a copy of the full output to the OS temp dir as `disputes-<slug>-<UTC-timestamp>.md` by actually calling a file-write tool. Print that path on one trailing line only after the tool call succeeds.

Group findings under Critical/Important/Minor only where it adds signal - don't force empty severity headings.
