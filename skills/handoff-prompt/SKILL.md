---
name: handoff-prompt
description: Compact the current conversation into a fenced handoff prompt in the reply, for pasting into a fresh agent.
argument-hint: "What will the next session be used for?"
disable-model-invocation: true
---

Reply with one handoff prompt in a fenced block, addressed to a fresh agent that has none of this conversation. Write no file.

Open the fence with four backticks (````markdown) so any code fences inside survive the copy. Put nothing outside the fence except a one-line note, if one is needed.

The prompt, in order:

1. **Goal**: what the next session does. If the user passed arguments, they set this; tailor everything below to it.
2. **State**: decisions made, work done, open questions, dead ends. Keep the conclusions and the reasons behind them.
3. **Pointers**: specs, plans, ADRs, issues, commits, diffs. Reference by path or URL; the fresh agent reads them itself.
4. **Suggested skills**: skills the fresh agent should invoke first.

Redact API keys, passwords, and personal data.
