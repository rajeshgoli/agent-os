# Shared Protocols

Essential rules all roles follow. Full system details in `director.md`.

## Session Start

Before any task, read the project's purpose doc if one exists (e.g., `.claude/why.md`) to calibrate quality expectations.

## Ownership

Every task has exactly one owner. Ownership transfers explicitly with **Instruction**.

## Handoff Checklist

- [ ] Work documented
- [ ] Next step is concrete
- [ ] Owner artifact updated
- [ ] Instruction provided (actionable, self-contained)

## Escalation

| Blocker | Escalate To |
|---------|-------------|
| Technical ambiguity | Architect |
| Product intent unclear | Product |
| Scope/priority conflict | User (via Product) |
| **Process friction** | **Director** |

### Process Issue Format
```markdown
## Process Issue for Director

**Role:** [Engineer/Architect/Product]
**Issue:** [Specific friction]
**Suggested Fix:** [Optional]
```

## Review Label (needs-review)

- Engineer: Add `needs-review` label + comment (summary + commit hash) when code ready
- Architect: Query `gh issue list -l needs-review` to find ready-to-review issues
- Architect: Review commit, add outcome comment (Accepted / Accepted with notes / Rejected), remove label when done
- Product/Director: Do NOT add or modify `needs-review` label

## Working Doc Lifecycle

**Working docs are the active design space**, not drafts. During investigation and design phases:

- Investigation/findings documented in markdown (no speculation, no theory)
- Architect/Product provide feedback via inline [Name] comments
- Multiple roles iterate via comments on the same doc
- Spec review happens HERE before tickets are created
- Working doc graduates to ticket/state doc only after closed-loop design

**Purpose:** Preserve design context across sessions, enable inline feedback, keep design iteration separate from execution planning.

## PR Comment Signature

Every PR comment must end with a signature line identifying the agent:

```
— <friendly-name> <sm-id> (<provider>)
```

Get your `sm-id` from `sm me` at session start. Example: `— architect-2570 abc123 (claude)`

## Memo Discipline (Lede-First)

Applies to any memo a role writes (specs, audit findings, retros, handoffs, research notes). Layered on top of the narrative-prose rule — this governs information order inside prose, not bullets-vs-prose.

- **Document-level lede.** The most important idea goes at the top. A reader who stops after the first paragraph should already know the verdict or main claim.
- **Section-level lede.** The first sentence or paragraph of each section states the core claim plainly. Justifications, footnote citations, and supporting detail come **after**.
- **Answer framing questions plainly first.** If the memo answers specific scope questions (from a parent doc, a ticket, or a dispatch prompt), the plain answer goes at the top of the relevant section. Don't bury the answer under setup.
- **Simple active sentences.** Split any sentence with 3+ commas or compound clauses. Prefer "X does Y" over "X, which does Y, is Z."
- **One idea per paragraph.** More paragraph breaks than you think.
- **Bullets for enumerations.** Fields, options, capabilities, sources — not comma-chained prose.
- **Verdicts and gate outcomes surface at the section top** with sharp one-sentence answers.

## Anti-Patterns

❌ Ambiguous ownership
❌ Archaeology required to understand state
❌ Unreviewed issues accumulating in `needs-review` label
❌ User over-query (explore with Architect first)
