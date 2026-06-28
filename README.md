# fslick/skills

Agent skills for a cross-model adversarial review loop.

## Install

```bash
npx skills add fslick/skills
```

## Skills

### `request-plan-review`
Runs in a **reviewer** window. Reviews an implementation plan as an independent
adversary and emits a structured report.

### `request-execution-review`
Runs in a **reviewer** window. Reviews code changes or an executed plan as an
independent adversary and emits a structured report.

### `address-review`
Runs in the **plan/code owner** window. Verifies each finding against reality, implements
accepted ones, and emits a `## Re-Review` block of disputed and uncertain findings
to paste back for the next round.

## Usage

1. Reviewer window: `/request-plan-review <plan-path>` or `/request-execution-review`
2. Owner window: `/address-review` with the report pasted in
3. Paste the `## Re-Review` block back into the reviewer window to iterate

Multiple reviewer models are supported: run `/request-plan-review` or
`/request-execution-review` in several windows (different models), then pass all
reports to one `/address-review` - it de-duplicates findings automatically.
