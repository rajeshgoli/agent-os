# agent-os

Shared agent instructions for Rajesh's supporting repos: **deskbar**, **office-automate**,
**finviz**, and **session-manager**. Works across Claude Code and Codex.

Everything is in one file: [`agents.md`](agents.md). It covers how work runs in these repos — the
work loop, the review loop, teardown, how to write for the owner, and the one way session-manager
differs. Each repo's own `AGENTS.md` carries what is specific to it.

There are no persona files. Modern models do not need a character sheet to act as an engineer or
a reviewer; what they need is the policy, and policy lives in `agents.md`. The persona directory
was removed in the rewrite — see git history if you need what it said.

## Setup

Add as a submodule:

```bash
git submodule add git@github.com:rajeshgoli/agent-os.git .agent-os
```

Then point the repo's `AGENTS.md` at it, above the project-specific content:

```markdown
Read `.agent-os/agents.md` for how work runs in this repo.

# [Project-specific content below]
```

A repo whose root `AGENTS.md` is auto-loaded still needs that first line, because the submodule
file is not loaded automatically — the agent has to go read it.

## The main project does not use this

`fractal-algo-rust` carries its own self-contained `AGENTS.md` at its repo root, with no submodule
and no indirection. Its workflow — parallel agents, epics, waves, review gates — is a different
shape from the small single-agent tickets these repos run on, and merging the two produced a file
that fit neither.

The `research` branch here served that project before the split and is superseded.
