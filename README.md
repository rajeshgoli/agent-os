# agent-os

Workflow system for AI coding agents. Provides personas, review protocols, and conventions that work across Claude Code and Codex.

## Branches

| Branch | Philosophy | Use when... |
|--------|-----------|-------------|
| `main` | **Infra mode** — fix everything, sequential epics, full test coverage | The codebase is stable infrastructure. Correctness is paramount. |
| `research` | **Research mode** — strategy docs, validation gates, parallel execution, speed over elegance | The codebase is exploratory. You're discovering what to build. |

Repos select a branch via git submodule tracking (see Setup below).

## What's in here

- **agents.md** — Core workflow instructions (attribution, review protocol, role invocation, ticket conventions, document types)
- **personas/** — Role definitions for agent specialization

## Personas

### Both branches

| Role | File | Purpose |
|------|------|---------|
| Engineer | `personas/engineer.md` | Implement from specs, write tests, create PRs |
| Architect | `personas/architect.md` | Review PRs, maintain architectural vision |
| Product | `personas/product.md` | Surface needs, interview users, set direction |
| Scout | `personas/scout.md` | Investigate bugs, write specs (no fixes) |
| UX | `personas/ux.md` | Review UI specs for consistency and completeness |
| EM | `personas/em.md` | Orchestrate multi-agent workflows |
| Director | `personas/director.md` | Maintain the workflow system itself |

### Research branch only

#### Software track (spec review)

| Role | File | Purpose |
|------|------|---------|
| Orchestrator | `personas/orchestrator.md` | Track owner (EM + main engineer). Owns strategy doc and execution doc, delegates aggressively, interfaces with user at validation gates. |
| Reviewer | `personas/reviewer.md` | Pure diff review — checklist, spec adherence, functional verification, merge. Does not write code. |
| Spec Owner | `personas/spec-owner.md` | Write and own specs. PR-based review — iterate with reviewer in the PR. Eliminate ambiguity before execution. |
| Spec Reviewer | `personas/spec-reviewer.md` | Review specs via PR comments. Verify claims against real code. Squash merge on convergence. |

#### Research track (collaborative science)

| Role | File | Purpose |
|------|------|---------|
| Applied Scientist | `personas/applied-scientist.md` | Holds the pen on research notes. Forms hypotheses, runs experiments, collects data, writes analysis scripts. Output is a mini paper, not a spec. |
| Peer Scientist | `personas/peer-scientist.md` | Collaborates — not just reviews. Runs own experiments, reproduces results, proposes alternatives, owns falsification and random controls. Can push to the PR. |

## Document Types (research branch)

| Type | Location | Lifecycle | Purpose |
|------|----------|-----------|---------|
| Strategy doc | `docs/product/` | Living — updated after each validation gate | North star, current plan, lessons learned |
| Execution ticket | `docs/working/` | Throwaway — replaced after each deliverable | Scoped spec for one deliverable (3-4 hours, up to 6 agents) |
| Research note | `docs/working/` | Throwaway — mini paper with reproducible artifacts | Hypothesis, methodology, evidence, robustness, conclusion |

## Review Protocols (research branch)

Two distinct protocols for two types of work:

**Software track — Spec Review (PR-based).** Spec owner creates a PR with the spec targeting dev. Reviewer posts feedback as PR comments. All iteration happens in the PR. On convergence, reviewer adds a Review History section and squash merges. `sm send` is used only as wake-up signals.

**Research track — Collaborative Science (PR-based).** Applied scientist opens a PR with a research note. Peer scientist collaborates — running experiments, contributing ideas, challenging conclusions, pushing code to the branch. Either scientist can kill a falsified hypothesis. On convergence, peer adds a Collaboration Log and squash merges. Failed experiments are documented and merged — they update the strategy doc.

## Setup

### Claude Code (local)

Symlink so Claude Code auto-loads the workflow for all projects:

```bash
ln -sf ~/.agent-os/agents.md ~/.claude/CLAUDE.md
```

Project-specific instructions stay in each project's `CLAUDE.md`.

### Codex (in-repo)

Add as a submodule. Use the `branch` option to select the philosophy:

```bash
# Infra mode (default)
git submodule add git@github.com:rajeshgoli/agent-os.git .agent-os

# Research mode
git submodule add -b research git@github.com:rajeshgoli/agent-os.git .agent-os
```

Then in your project's `AGENTS.md`:

```markdown
Read .agent-os/agents.md for workflow instructions and persona definitions.

# [Project-specific content below]
```

### Switching branches for an existing submodule

```bash
git config -f .gitmodules submodule..agent-os.branch research
git submodule update --remote .agent-os
```

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
