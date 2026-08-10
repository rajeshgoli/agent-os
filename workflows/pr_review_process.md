# PR Review Process — moved

This process now lives in [`agents.md`](../agents.md), section 3 ("The review loop"). Read it
there.

This file remains because the review process is often invoked by name — "use
`.agent-os/workflows/pr_review_process.md`" — and an instruction that dead-ends costs more than a
forwarding note.

In short: request with `sm request-codex-review <pr-number>`, go idle rather than polling, confirm
the review you read belongs to your current request, classify every finding as valid, partially
valid, or invalid, treat any unresolved P1 as blocking, and exit on a round that returns only P2
or lower. `agents.md` §3 has the fallback path and the rest.
