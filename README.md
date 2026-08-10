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

For Claude Code, the repo also needs a root `CLAUDE.md` carrying the same pointer, so it is picked
up regardless of which filename the running version auto-loads. **Check what is already there
first — an older layout kept project-specific rules in `CLAUDE.md`, and overwriting it loses
them.**

```bash
ls -l CLAUDE.md          # symlink, real file, or absent?
```

- **Absent** — create the symlink: `ln -s AGENTS.md CLAUDE.md`
- **Already a symlink to `AGENTS.md`** — nothing to do.
- **A real file** — read it. Anything in it that `AGENTS.md` does not already say has to be merged
  into `AGENTS.md` first. Only once the content is in `AGENTS.md` may the file be replaced with the
  symlink. Never `ln -sf` over it; `-f` deletes the destination.

As of this writing `session-manager/CLAUDE.md` is a real file of about 140 lines that its
`AGENTS.md` does not contain, so that repo needs the merge before it needs the symlink.

**Do not symlink this into `~/.claude/CLAUDE.md`.** The old setup did that to apply the workflow
to every project at once. It is now wrong: these rules describe small single-agent tickets, and a
global symlink would apply them to `fractal-algo-rust`, which has its own and very different
contract. Wire it per repo.

If a machine still has that symlink from the old setup, it needs removing — not creating it is not
the same as it being gone, and a stale one keeps loading this contract into every repo including
the one that must not have it. Check what is there:

```bash
readlink ~/.claude/CLAUDE.md      # empty means absent or a real file — either way, leave it
```

Delete it **only** if it resolves to the old `~/.agent-os/agents.md`. Anything else — a real file,
or a symlink to some other configuration — is someone's live setup and is not this migration's to
touch.

Removing it is left as a manual step on purpose. A copy-pasteable `rm` guarded by a pattern is how
this instruction went wrong twice: first an unconditional `rm` on the line after its own check,
then a glob that also matched paths like `~/custom-agent-os-config/`. The check above is
read-only, and one person deleting one symlink deliberately is safer than any guard written for a
machine state nobody has confirmed.

## The main project does not use this

`fractal-algo-rust` carries its own self-contained `AGENTS.md` at its repo root, with no submodule
and no indirection. Its workflow — parallel agents, epics, waves, review gates — is a different
shape from the small single-agent tickets these repos run on, and merging the two produced a file
that fit neither.

The `research` branch here served that project before the split and is superseded.
