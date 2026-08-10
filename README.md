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

That first line is required. The submodule file is never loaded automatically — the agent has to
be told to go read it.

For Claude Code, give the repo a root `CLAUDE.md` carrying the same content, so the pointer is
picked up regardless of which filename that version auto-loads:

```bash
ln -sf AGENTS.md CLAUDE.md
```

**Do not symlink this into `~/.claude/CLAUDE.md`.** The old setup did that to apply the workflow
to every project at once. It is now wrong: these rules describe small single-agent tickets, and a
global symlink would apply them to `fractal-algo-rust`, which has its own and very different
contract. Wire it per repo.

## The main project does not use this

`fractal-algo-rust` carries its own self-contained `AGENTS.md` at its repo root, with no submodule
and no indirection. Its workflow — parallel agents, epics, waves, review gates — is a different
shape from the small single-agent tickets these repos run on, and merging the two produced a file
that fit neither.

The `research` branch here served that project before the split and is superseded.
