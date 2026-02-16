# agent-os

Workflow system for AI coding agents. Provides personas, review protocols, and conventions that work across Claude Code and Codex.

## What's in here

- **agents.md** — Core workflow instructions (attribution, review protocol, role invocation, ticket conventions)
- **personas/** — Role definitions (engineer, architect, product, scout, ux, em, director)

## Setup

### Claude Code (local)

Symlink so Claude Code auto-loads the workflow for all projects:

```bash
ln -sf ~/.agent-os/agents.md ~/.claude/CLAUDE.md
```

Project-specific instructions stay in each project's `CLAUDE.md`.

### Codex (in-repo)

Add as a submodule so Codex can read the workflow:

```bash
git submodule add git@github.com:rajeshgoli/agent-os.git .agent-os
```

Then in your project's `AGENTS.md`:

```markdown
Read .agent-os/agents.md for workflow instructions and persona definitions.

# [Project-specific content below]
```

## Personas

| Role | File | Purpose |
|------|------|---------|
| Engineer | `personas/engineer.md` | Implement from specs, write tests, create PRs |
| Architect | `personas/architect.md` | Review PRs, maintain architectural vision |
| Product | `personas/product.md` | Surface needs, interview users, set direction |
| Scout | `personas/scout.md` | Investigate bugs, write specs (no fixes) |
| UX | `personas/ux.md` | Review UI specs for consistency and completeness |
| EM | `personas/em.md` | Orchestrate multi-agent workflows |
| Director | `personas/director.md` | Maintain the workflow system itself |

## How it works

```
┌─────────────────────────────┐     ┌──────────────────────────────┐
│  ~/.agent-os/agents.md      │     │  project/CLAUDE.md           │
│  (generic workflow)         │     │  (project-specific)          │
│                             │     │                              │
│  - Review protocol          │     │  - Dev commands              │
│  - Role invocation          │     │  - Data formats              │
│  - Ticket conventions       │     │  - Branch rules              │
│  - Attribution policy       │     │  - Debugging methodology     │
└─────────────┬───────────────┘     └──────────────┬───────────────┘
              │                                    │
              │  symlink                           │  auto-loaded
              ▼                                    ▼
        ~/.claude/CLAUDE.md              project/CLAUDE.md
              │                                    │
              └────────────┐  ┌────────────────────┘
                           ▼  ▼
                    Claude Code agent
                   (sees both files)
```

Codex gets the same content via `.agent-os` submodule + `AGENTS.md`.
