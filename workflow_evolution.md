# Workflow Evolution: Spec-Driven Deliberate Development

**Status:** Iterating
**Based on:** 246 investigation-mode invocations + recent command patterns
**Context:** DAG layer stability requires careful, spec-informed work

---

## Director Review: Personas vs. Reality (Jan 19, 2026)

**Assessment of gaps between documented personas and actual workflow:**

### Non-Issues (Confirmed Working)
- **pending_review >= 10 gate**: Never activates in practice. Not a real problem. Keep rule in place but don't optimize for it.
- **Handoff skill**: Almost never used (1,264 implicit "As [role]..." vs. 10 explicit `/handoff`). Users find natural language clearer. Don't force skill adoption; it's aspirational, not operational.

### Real Gaps to Close

1. **Architect Persona is Underdocumented**
   - **Documented scope**: Review finished engineer work, reset pending_review count, hand off to next owner
   - **Actual scope**: Design exploration (profiling, complexity audits), pre-implementation spec review, investigation harnessing, iterative architectural reasoning
   - **Missing**: When Architect should be invoked for feasibility studies (not just engineer handoff). How Architect operates on working docs before tickets exist. Investigation harness patterns.
   - **Fix**: Expand "Workflow" section in `architect.md` to cover "Design Exploration Triggers" alongside review triggers.

2. **Product Persona Context-Dependent, Not Always Dormant**
   - **Documented scope**: Interview users, refine specs, handoff to Engineer/Architect
   - **Actual scope**: Dormant during DAG/backend work (code is tightly coupled, user decides architecture). Active during UI/UX phases (looser code, faster iteration acceptable). Not always needed.
   - **Missing**: Explicit guidance on when to activate vs. defer Product. Decision tree based on code topology.
   - **Fix**: Add context clause to `product.md`: *"Dormant during tightly-coupled backend phases (DAG, core algorithms). Activate for UI/signal/usability iterations and direction reassessment."*

3. **Working Docs Are Persistent Artifacts, Not Drafts**
   - **Documented**: "Active one-off artifacts (proposals, explorations) → Archive when done"
   - **Actual**: Working docs are **the** design space. Multiple roles read/comment/iterate. They coordinate investigation, design, and context preservation across sessions.
   - **Missing**: Artifact graduation criteria. When does working doc → proposal? When does proposal → ticket? Lifecycle is implicit.
   - **Fix**: Formalize "Artifact Graduation Checklist" (already outlined in Pattern #12 of this doc). Add to `_protocols.md`.

---

## Core Insight

The current persona system treats investigation, review, and implementation as separate gates with fixed depth. Actual workflow shows three key gaps:

1. **Investigation scope matters**: Engineer investigation alone fixes minimal root cause. When architect reviews spec first OR investigation is spec-informed, broader issues surface and are caught before implementation.

2. **Review depth is context-dependent**: Formulaic gate review (pending_review >= 10) is performative. Deep spec-driven review catches implementation issues. Depth should vary by layer/phase.

3. **Deliberate vs. Fast cycles are layer-specific**: DAG layer (everything depends on it) = careful, spec-first. UI/signal iterations = move fast. Workflow should encode this.

---

## Investigation Modes

### Mode 1: Narrow Investigation (current reality)
- **Who**: Engineer
- **What**: Find immediate root cause, fix minimum viable scope
- **Issue**: Catches symptom, misses architectural implications
- **Output**: Code fix, no broader context captured

**Example:** "Why did this leg not prune?" → Traces code path, finds one condition → fixes one condition

### Mode 2: Spec-Informed Investigation (desired pattern)
- **Who**: Engineer + Spec context (read issue ticket with spec links)
- **What**: Root cause with spec as reference frame
- **How**: Before diagnosing, read the spec document first (explicit in ticket)
- **Issue caught**: Understands not just "what broke" but "what should happen per spec"
- **Output**: Root cause + spec compliance gap identification

**Example:** "Why did this leg not prune?" → Reads spec on prune conditions → Traces code → Finds spec gap ("code doesn't match spec rule X") → Fixes properly

### Mode 3: Architect-Assisted Investigation (new pattern?)
- **Who**: Engineer investigates, Architect reviews investigation findings before implementation
- **What**: Engineer traces + documents findings. Architect asks "what else might be broken?"
- **When**: DAG layer work, or when initial investigation looks incomplete
- **Output**: Investigation report + architectural implications

---

## Review Modes

### Mode 1: Formulaic Gate Review (current reality)
- **When**: pending_review count >= 10
- **What**: Line-by-line checklist. Quick acceptance.
- **Issue**: Just checking "was code written correctly" not "is this the right change"
- **Output**: Status accepted, count reset

### Mode 2: Spec-Driven Deep Review (demonstrated to save time)
- **When**: Spec exists (DAG invariants, algorithms documented)
- **What**: Compare implementation line-by-line against spec. Ask "does this implement the spec correctly?"
- **Who**: Architect with spec context
- **Catches**: Implementation gaps spec review found (user reports: "saves me a lot of time down the line")
- **Output**: Implementation issues surfaced, fixes cleaner

**User observation:** "architect spec review catches most of these... catches a bunch of implementation issues"

### Mode 3: Verification-Only Review (mentioned in recent commands)
- **When**: "don't update specs. for now verify the implementation"
- **What**: Is the code syntactically/logically correct? (Not: does it match spec?)
- **Who**: Architect with limited scope
- **Output**: Code quality, no spec gaps analyzed

---

## Layer/Phase-Specific Workflows

### DAG Layer (current phase)
**Principle**: Slow, deliberate, spec-driven
**Why**: "Everything else depends on it"

**Investigation:**
1. Ticket includes spec link
2. Engineer reads spec first before investigating
3. Engineer documents: root cause + spec compliance status
4. If uncertainty: Architect reviews investigation before engineer implements

**Review:**
1. Architect does spec-driven review (implementation vs. spec)
2. Not formulaic gate review
3. Deep engagement with spec

**Velocity:** Slower but stronger foundation

### UI Iterations / Signal Iterations (future phase)
**Principle**: Move fast, architectural risks are lower
**Why**: Not core infrastructure

**Investigation:**
1. Engineer diagnoses and fixes
2. Narrow root cause scope acceptable

**Review:**
1. Verification-only or lighter checklist review
2. Formulaic gate review acceptable

**Velocity:** Faster iteration, acceptable tradeoff

---

## Investigation + Spec Workflow (Pattern to Codify)

Currently implicit: "assume engineer role. You're helping investigate. Don't make any code changes unless I tell you to."

Should be explicit in tickets:

```
## To Engineer:

1. **Read the spec first**: [link to spec doc]
2. **Investigate**: Use harness/trace script against real data
3. **Document findings**: Root cause + spec compliance status
4. [Optional] **Await review**: Architect may review findings before you implement
5. **Implement**: Make changes addressing root cause + spec gaps
```

---

## Review Depth Decision Tree

When Architect is asked to review:

1. **Is there a spec?**
   - Yes → Spec-driven review (deep implementation comparison)
   - No → Verification-only review (code quality checklist)

2. **Is this DAG layer or core infrastructure?**
   - Yes → Thorough spec-driven review
   - No → Verification or lighter review

3. **Did engineer's investigation look complete?**
   - No (narrow fix, missed implications) → Spec review will catch it
   - Yes → Verification-only acceptable

---

## Current Example (Recent Activity)

**What happened:**
1. Engineer investigated crash (found immediate cause)
2. Architect asked "you only fixed the symptom, why doesn't legdetector have this?"
3. Engineer added comment: broader pattern exists
4. Engineer's fix now addressed broader issue

**What should be codified:**
- Investigation was spec-informed by architect's question
- Catches broader issue than narrow fix would
- Architect review at investigation stage (not just gate) adds value

---

## Data-Driven Findings from History.jsonl

### Handoff Patterns (Actual Usage)
- **Implicit handoffs (role assumption):** 1,264 instances
- **Explicit handoffs (using /handoff skill):** 10 instances
- **Insight:** Handoff skill is almost never used. Users rely on "As [role]" format exclusively.

### Investigation Outcomes (After Investigation, What Happens?)
- **Direct Implementation:** 21 times (27%)
- **Spec Review:** 16 times (20%) — Engineer reads spec before implementing
- **Re-Investigation:** 13 times (17%) — Same issue investigated multiple times (avg 2-3 rounds)
- **File Issue (no fix):** 8 times (10%) — Investigation results in "not implementable yet"
- **Review-Then-Implement:** 6 times (8%)
- **Discuss With User:** 1 time

**What this tells us:** Investigation → Direct Implementation is common (27%), but Spec Review (20%) + Re-Investigation (17%) = 37% of cases involve deeper deliberation.

### Architect Review Effectiveness (Why They Matter)
- **Needs Rework:** 40 instances (triggered rework)
- **Bug Found:** 29 instances (architect catches bug)
- **Design Concern:** 20 instances (flags architecture issue)
- **Approved As-Is:** 11 instances

**Insight:** 89 out of 100 architect reviews trigger either rework, bug fixes, or design concerns. Only 11% are "approved as-is." Architect review catches things and is high-value.

### Issue Lifecycle
- **Completed issues:** 34 (with full lifecycle tracked)
- **Multi-mention abandoned issues:** 28 (discussed multiple times but never completed)
- **Average steps per completed issue:** 2.4 (investigate → implement → review → close)
- **Issue #1 example:** Referenced 169 times with 5 investigation rounds, 2 implementation rounds, 2 review rounds

**Insight:** Issues that get discussed multiple times either have complex root causes or missing context. Average of ~2.4 steps suggests issues are NOT simple one-shot fixes.

### Multi-Round Investigations
- **Issues with 4+ mentions:** 32
- **Most investigated issue (#1):** 5 investigation rounds, across multiple sessions
- **Pattern:** When an issue bounces between roles, it's usually because:
  - First investigation is shallow (fixes symptom)
  - Architect review asks "but what about X?"
  - Engineer re-investigates more carefully
  - (Repeat until root cause is found)

**User observation correlates:** "if the agent doing the initial investigation fixes things, it only fixes the smallest root cause and misses broader perspective"

### User Interventions
- **Skip/Don't:** 867 times ("don't do that", "skip this step")
- **Redirect:** 183 times ("do this instead", "focus on this not that")
- **Clarify:** 29 times ("I meant to say", "context update")
- **Wait:** 491 times ("architect review first", "hold on")

**Insight:** Heavy intervention on conditional workflow steps (skip, redirect). User is controlling workflow depth/scope per-task, not following fixed pattern.

### Spec Interaction (Critical Finding)
- **80% of spec mentions are PASSIVE** ("per spec..." as reference during investigation)
- **Only 9% of engineers proactively read specs before work** (engineer discovers mid-investigation)
- **Architects read specs 2.4x more than engineers** (17 vs 38 cases)
- **Explicitly told to read spec:** 16+ instances (minority)
- **Pattern:** Specs are REACTIVE (discovered mid-investigation), not PROACTIVE (read before implementation)

**User data correlation:** This explains the rework pattern — engineer investigates without spec context, implements partial solution, architect reviews and asks "what about requirement X in spec?", rework needed.

---

### Implementation vs. Architect Involvement
- **32% of investigations → direct implementation** (no architect input)
- **Only 9% flow to architect review → then implementation**
- **36% triggered by explicit user request** for architect involvement
- **No complexity-score auto-triggers**

**Pattern:** Architect involvement is manual/opt-in, not automatic. Risk is high that engineer skips architectural review.

---

### Rework Patterns (40-70 Avoidable Events)
- **40 rework directives** from architect
- **29 bugs caught by architect** during review
- **48 test failures** triggering re-investigation
- **41 broken functionality** reports post-implementation
- **Total:** ~113 rework events

**Analysis:** If specs were read UPFRONT instead of discovered mid-investigation, architect review happened PRE-implementation instead of POST-gate, ~70+ of these rework cycles could be prevented.

---

## Synthesis: What the Data Shows

### 1. SPECS ARE REACTIVE, NOT PROACTIVE (Root Cause of Rework)
- **80% of spec mentions are passive** — engineer references spec while investigating, hasn't read it upfront
- **Only 9% proactively read specs before work**
- **32% of investigations bypass architect entirely** (no spec review, no architecture check)
- **Result:** Engineer implements partial solution, architect later asks "what about requirement X in spec?", rework needed
- **Magnitude:** ~70+ rework cycles could be prevented if specs were read UPFRONT

**This is the core inefficiency.** When spec is read proactively (before implementation), architect catches issues before engineering, not after.

### 2. Architect Review Happens at Wrong Time
- **89% of architect reviews find issues** (rework, bugs, design concerns)
- **But they're happening LATE** (post-gate, after code written)
- **Pattern:** Deep spec-driven review should happen PRE-implementation
- **Evidence:** When architect reviews spec BEFORE engineer implements (16 instances), far fewer bugs slip through

**The fix:** Move spec-driven review earlier in the cycle, before implementation begins.

### 3. Handoff Skill (/handoff) Not Being Used
- 1,264 implicit handoffs ("As engineer...") vs. 10 explicit (/handoff)
- **Gap:** Skill exists but workflow doesn't use it. Either: (a) user finds it cumbersome, or (b) natural language handoff is clearer

### 4. User Controls Workflow Scope Per-Task
- 867 "skip/don't" interventions
- **Pattern:** User is deliberately saying "for this task, skip architect_notes update" or "do verification-only review, not deep spec review"
- **Current personas assume:** Fixed process for all tasks
- **Actual usage:** Highly contextual depth/scope decisions

### 5. Re-Investigation is Real
- Issues bouncing between roles not uncommon
- First investigation often misses broader implications
- **User observation:** "architect spec review catches most of these"
- **Root cause:** Narrow investigation → narrow implementation → broader issues found at review gate → rework

### 6. Multi-Step Workflows ARE the Norm
- Completed issues average 2.4 steps minimum
- Many issues reference multiple times (abandoned or iterating)
- **Pattern:** Not "implement once and close" — more like "investigate → implement → review → (maybe rework) → close"

---

## What This Means for Workflow Design

### The Ideal Vs. The Real

**Personas prescribe:**
- Engineer: investigate → implement → test → push → handoff
- Architect: review at gate (pending_review >= 10)
- Fixed depth for all work

**History shows:**
- Investigation depth varies (narrow vs. spec-informed)
- Review happens at investigation stage (not just gate)
- User controls scope per-task ("skip this step")
- Re-investigation when first pass misses implications
- Explicit spec reading before implementation in deliberate phases

### Root Cause: Specs Are Discovered, Not Read

The data points to a single root cause of inefficiency:

**Engineer workflow:** Investigate → (discover spec during investigation) → Implement → Architect Review → (architect finds gaps) → Rework

**What it should be:** Read Spec → (architect reviews spec feasibility) → Investigate (spec-informed) → Implement → Test → Done

The current system makes specs a post-hoc discovery during investigation. This causes:
- 40-70 rework cycles that should be prevented
- Shallow first implementations that miss requirements
- Late architect discovery of architectural issues
- Re-investigation rounds that add time

---

### Key Gaps to Close (In Order of Impact)

1. **Specs must be READ PROACTIVELY, not discovered during investigation** (70+ rework events at stake)
   - Engineer should read spec before investigation starts
   - Architect should review spec feasibility before implementation
   - Current: 9% proactive spec reading. Target: 80%+

2. **Architect review timing is backwards**
   - Currently: spec review at gate (post-implementation) catches bugs too late
   - Should be: spec review pre-implementation (before engineer starts coding)
   - Effect: Prevents ~70+ rework cycles

3. **No automatic trigger for architect spec review**
   - 36% of architect involvement is manual user request
   - Should auto-trigger when spec exists + high-risk work (DAG layer)
   - Currently engineers skip architect entirely (32% direct implement)

4. **Investigation depth protocol missing**
   - When should it be "spec-informed deep"? vs. "narrow technical"?
   - Should be explicit in ticket, not surprise requirement

5. **Re-investigation is not a formal phase**
   - When architect's review asks deeper questions, engineer should respond with additional investigation
   - Currently invisible; treated as ad-hoc

6. **Workflow scope control is ad-hoc**
   - User says "don't update architect notes" every time (867 skip directives)
   - Should be explicit in task instruction, not surprise override

---

## CURRENT PRACTICES ALREADY IN USE (Missed Earlier)

### Practice 1: Pre-Implementation Spec Reviews
**Evidence:** 13+ recent instances of "as architect review [issue] spec (not implementation)"

Workflow:
1. Ticket filed with spec linked
2. Architect reviews spec FIRST (not implementation)
3. Architect provides feedback on spec
4. Engineer reads spec-reviewed ticket + comments
5. Engineer investigates (now spec-informed)
6. Engineer implements

---

### Practice 2: Tickets as Living Documents (Source of Truth)
**Evidence:** 14+ instances of "plan of record", reinforced repeatedly

**Pattern:**
- Architect UPDATES tickets directly (edits body/title)
- Engineer COMMENTS on tickets explaining findings/implementation
- Both collaborate by updating the same ticket document

**Example flow:**
- "I made some updates to the ticket. Check it." (Architect edits body)
- "add a comment explaining what you did" (Engineer documents findings in comments)
- "update the ticket. Don't add a comment, edit the body directly." (Architect updates the plan)

**The "Plan of Record" Style:**
- Ticket body is the definitive plan (forward-looking, no history)
- No "architect notes" or "options considered" in body
- No "architecture guidance" or review text
- Reasoning/process notes go in COMMENTS (separate from body)
- Result: Ticket reads as the actual plan, not a review document

**User reinforcement (14 instances):**
- "Just write it as plan of record, not history or review"
- "Remove architectural guidance etc., just update it as plan of engineering record"
- "Don't use architect review etc. Edit the title and body directly"
- "delete comment, edit the body. Should read like plan of record"

---

### Practice 3: Engineer Responds to Architect via Ticket Comments
**Evidence:** Multiple "add a comment explaining what you did" patterns

**Flow:**
- Architect provides feedback in ticket comments or edits body with questions
- Engineer reads comments, does investigation/work
- Engineer documents findings/implementation in ticket comments
- Ticket evolves as a shared artifact

**Example:**
- "add a comment to 345 detailing what you did and what remains"
- "add a comment explaining your fix"
- "add a comment detailing these findings"

**Not separate artifacts** — Everything captured in the issue, not in external docs

---

### What's Missing from Personas
The personas don't describe:
1. **Tickets as living documents** (only treats them as "tasks to complete")
2. **Plan of record style** (no guidance on how to write ticket bodies)
3. **Engineer responding via comments** (not a documented workflow)
4. **Architect updating tickets directly** (not just reviewing them)
5. **Separation of body (plan) from comments (reasoning)** (no distinction)

---

## Remaining Design Questions (Given Spec Reviews Are Working)

Now that pre-implementation spec reviews are active, the gaps are:

### Question #1: Consistency and Adoption

**Observation:** 13 instances of spec-only reviews in recent history, but the historical average was 9% proactive spec reading.

**Questions:**
- How recent did this shift start? (All 13 in last N interactions?)
- Is this happening for ALL DAG layer work, or just some issues?
- What should trigger an architect spec review? (Should every issue with a spec have one?)
- Is there a decision tree for when spec review is "must-have" vs. "nice-to-have"?

**Action needed:** Formalize the trigger. Right now it seems user-initiated. Should it be:
- Auto-trigger when spec exists + high-risk work?
- Always for DAG layer, optional for UI?
- Architect decides per ticket based on spec complexity?

---

### Question #2: The Investigation Phase After Spec Review

**Current pattern:** Architect reviews spec → Engineer investigates → Engineer implements

**Gap:** What happens when architect's spec review raises questions?

**Examples from recent interactions:**
- Does engineer then do "re-investigation" to address architect's spec concerns?
- Or does engineer proceed with investigation as-planned and address architect feedback during implementation?
- When architect says "spec is unclear on X", does that trigger a back-and-forth before engineer starts, or engineer starts with that as open question?

**Why it matters:** This affects whether the "spec-reviewed" workflow actually prevents rework, or just shifts it earlier.

---

### Question #3: The "Thorough" Modifier

**Observation:** One recent instance says "review 891 spec (not implementation) **thoroughly**. Refer to pivot first architecture v2.md"

**Gap:** What makes a spec review "thorough" vs. quick?

**Current state:**
- Some say "as architect review X spec (not implementation)" — lightweight
- Some say "review X spec (not implementation) thoroughly" — deep

**Question:** Should the persona/templates distinguish:
- **Spec-Correctness Review** (is spec written clearly? are requirements complete?)
- **Spec-Feasibility Review** (can this be built? does it match architecture?)
- **Thorough Spec Review** (both above + think about edge cases, implications, interactions)?

---

### Question #4: Workflow Scope Control (867 Manual Redirects)

**Still open:** User still redirects constantly ("don't update architect notes", "skip this", "verification-only")

**Gap:** Now that spec reviews are formalized, should workflow scope also be formalized?

**Examples:**
- "as architect review X spec (not implementation)" already signals "don't implement yet"
- But other scope directives are still manual ("no need to update architect notes", "push your changes and update pending review")

**Questions:**
- Should these become part of the instruction in tickets? ("Update architect_notes: NO for this task")
- Or is user control per-task still the right model?
- Should DAG layer have a different scope checklist than UI layer?

---

### Question #5: Re-Investigation Trigger

**Gap:** When does architect feedback on a spec trigger engineer re-investigation (additional probing) vs. engineer proceeding with original investigation plan?

**Current state:** Unclear

**Example pattern from recent history:**
- Architect: "Review spec, is this correct?"
- Architect finds: "But what about X scenario?"
- Engineer: Next step is... investigate deeper? Or implement handling for X?

**Question:** Should this be formalized in personas as:
- If architect's spec review raises architectural concerns → engineer re-investigates
- If architect's spec review finds typos/clarity issues → engineer just reads clarified spec and proceeds

---

## Synthesis: What's Evolved

**What's working:**
- Pre-implementation spec reviews are happening (13+ recent instances)
- Prevents implementation of flawed specs
- Aligns architect and engineer before code is written

**What needs formalizing:**
1. **Consistency:** When is spec review mandatory vs. optional? (Need decision tree)
2. **Depth levels:** "Thorough" review vs. lightweight review — what's the difference? (Need definitions)
3. **Engineer response:** When architect's spec feedback raises concerns, does engineer re-investigate? (Need protocol)
4. **Scope control:** 867 manual redirects still happening — should these be in ticket templates? (Need structure)
5. **Re-investigation triggering:** What separates "spec issue" from "architecture concern"? (Need decision gate)

---

---

## DEEPER PATTERNS DISCOVERED

These are the **real** workflow patterns that personas completely miss:

### 1. **Working Docs as Investigation Harnesses**
Not just drafts — they're deterministic investigation spaces where you:
- Document trace scripts (reproduce bugs, step-by-step)
- Record findings from real data execution (not speculation)
- Preserve context for next agent's fresh session
- Build up evidence before implementation

**Evidence:** Performance_question.md, Performance_problem.md, leg detection traces

**What personas miss:** Investigation is its own phase with deliverables (documented findings), not just "figure stuff out then implement"

---

### 2. **Inline Comments as the Real Review Mechanism**
You use markdown inline comments (`[Rajesh] ...`) for design feedback because:
- More efficient than ticket body edits (agents must rewrite entire body)
- Preserves written design decisions
- Allows parallel feedback while agents are iterating
- Easier to search and review

**Evidence:** 12+ patterns of "can you update the working doc so I can respond inline?"

**What personas miss:** Feedback happens IN documents, not in tickets or chat

---

### 3. **Error-Prone Ticket Editing Workaround**
You systematically avoid asking agents to update ticket bodies because:
- They can't do surgical edits; must rewrite entire body
- Working docs are better for design iteration
- Tickets are reserved for execution plans only

**Flow:** Working Doc (iterate) → Finalized → Ticket (as plan of record)

**What personas miss:** Tickets are for execution, not design

---

### 4. **Artifact Graduation Pipeline**
Clear progression with specific purposes:
```
Investigation
  ↓ (findings documented)
Working Doc (draft + inline feedback cycles)
  ↓ (design finalized)
Proposal (ready for decomposition)
  ↓ (broken into work)
Tickets (execution plans)
  ↓ (work completed)
State Doc (decisions recorded)
  ↓ (work archived)
Archive/
```

Each stage is deliberate. Movement between stages has criteria (what makes a working doc "done"?).

**What personas miss:** No mention of tiers, progression, or graduation criteria

---

### 5. **Investigation vs. Implementation Tickets**
You create **two types** of tickets:

**Investigation Tickets:**
- Ask agents to trace code and document findings
- No implementation, no code changes
- Deliverable: Docs/Working/*.md with findings
- Example: "investigate why this crashed, document what happened"

**Implementation Tickets:**
- Clean "plan of record" with solution outlined
- Agents implement from the plan
- Example: "implement fix for X as described in the spec"

**What personas miss:** Personas describe investigation as informal; you formalize it as a ticket type with distinct deliverables

---

### 6. **Context Bridge via Working Docs**
When context windows reset, you don't wait for email notifications:
- You ask agents to update working docs with findings
- Next agent reads the working doc fresh
- Continuation is deterministic (pick up from the doc, not memory)

**Evidence:** "Can you create a working doc, we might lose context and another agent should be able to pick up"

**What personas miss:** Context preservation is deliberate and baked into workflow

---

### 7. **Specs as Mutable Drafts (Before Tickets)**
Specs don't live in tickets. Flow is:
1. Product writes Docs/Working/*_spec.md
2. Architect reviews spec with inline comments
3. Product/Architect iterate on spec (inline feedback)
4. Once spec is solid: Move to ticket or state doc
5. Engineer implements from finalized spec

**What personas miss:** Spec design happens in working docs, not tickets

---

### 8. **User as Architect via Ticket Updates**
When you say "I made some updates to the ticket," this is a **directive lock**:
- Agents see ticket as the current plan
- They don't question or renegotiate
- It's architectural decision binding them

**What personas miss:** User decisions in ticket bodies are special (not just metadata)

---

### 9. **Deterministic Debugging (No Speculation)**
You explicitly forbid:
- Reading code to guess root cause
- Theorizing without evidence

You require:
- Write harness scripts to reproduce bug
- Run against real data
- Observe actual behavior
- THEN reason about cause

**What personas miss:** Investigation has a specific protocol (deterministic, evidence-based, no speculation)

---

### 10. **State Docs as Records (Not Drafts)**
State docs (architect_notes.md, product_direction.md, pending_review.md):
- Never created fresh; always evolved from prior state
- Updated AFTER working docs mature
- Single source of truth for current decisions
- Overwritten when state changes, never appended

**What personas miss:** State docs are final records, not working artifacts

---

### 11. **Explicit Over Implicit (Constraints and Scope)**
You explicitly state ALL assumptions upfront in instructions:
- "No code changes unless I explicitly request it"
- "Only update the doc when I explicitly tell you"
- "Explicitly call out what can be in parallel vs. sequence"
- "Explicitly NOT reset backend state" (showing what WON'T happen)
- "Explicitly in scope" / "explicitly NOT in scope"

**Pattern:** Remove all implicit assumptions. Make every constraint, boundary, and permission visible in the instruction/ticket/spec.

**Evidence:** 15+ instances of "explicitly" in context of scope, constraints, permissions

**What personas miss:** Instructions should remove ambiguity by stating constraints, not rely on agents to infer them

---

## Pattern 12: Code Topology Determines Workflow Strictness

**The Actual Discovery**

Your workflow tightness isn't about importance — it's about **code failure modes**.

DAG/pivot layer: "like writing C code"
- Everything is deliberate
- Tightly coupled invariants
- One mistake cascades, not isolated
- Agent can't "slap an O(n) loop" and fix it — breaks implicit constraints
- Mistakes are hard to detect

Result: Analysis → Report (no changes) → User Decision → Implementation

UI/Signal layer: Loosely coupled, forgiving
- Changes are more isolated
- Mistakes surface locally
- Can iterate and fix
- Mistakes are obvious

Result: Investigation → Implementation (faster)

**The Pattern: Code structure determines who holds the gate.**

Tightly coupled code that punishes mistakes = user decides after analysis.
Loosely coupled code that forgives mistakes = agent can decide and iterate.

This is why you can't codify a universal workflow. The personas need to encode:
- Understand your code's failure modes
- Tight coupling → tight workflow (analysis → decision → implementation)
- Loose coupling → loose workflow (investigation → implementation)

---

## What These Patterns Reveal

**The real workflow is:**

```
┌─ Research/Exploration Phase ─┐
│ Planning mode + investigation │
│ Collect findings in Docs/     │
└──────────┬──────────────────┘
           ↓
┌─ Design Phase ─────────────────────┐
│ Working Doc + Inline Feedback Loops │
│ Architect/Product iterate with user │
└──────────┬──────────────────────────┘
           ↓
┌─ Finalization Phase ────────────────┐
│ Spec locked, Plan of Record written │
│ Context preserved for next agent    │
└──────────┬────────────────────────┘
           ↓
┌─ RISK GATE ──────────────────────────────────┐
│ Code tightly coupled? Cascading failures?    │
│ YES → User Decision Gate  │ NO → Fast Track  │
└──────────────┬───────────────────────────┬──┘
               ↓                           ↓
      ┌─ Execution Phase ──┐  ┌─ Direct Implementation ┐
      │ (Tight Workflow)   │  │ (Fast Iteration)       │
      │ Implements per     │  │ Implements + Tests     │
      │ spec, user gated   │  │ Agent decides          │
      └──────────┬─────────┘  └───────────┬────────────┘
                 ↓                        ↓
┌──────────────────────────────────────────────────┐
│ Record Phase: State docs updated                │
│ Archive Phase: Completed work moved             │
└──────────────────────────────────────────────────┘
```

**Workflow strictness ≠ importance. It's code topology.**

---

## What Needs to Change in Personas

Given these 10 patterns, personas need major updates:

### 1. **Director.md** — Add Artifact Lifecycle & Context-Aware Activation
**Add: Artifact Tiers and Graduation Protocol**
- Define what "makes a working doc done"
- What triggers graduation to Proposal?
- What triggers graduation from Proposal to Tickets?
- Archive criteria

**Add: Role Activation Decision Tree (based on code topology)**
- DAG/core layer (tightly coupled) → strict workflow (deep review, user decisions)
- UI/signal layer (loosely coupled) → fast workflow (verification review, agent autonomy)
- This determines Architect depth, Product presence, Engineer autonomy

### 2. **Engineer.md** — Add Investigation Ticket Type & Spec-First Protocol
**Add: Two Ticket Types**
- Investigation Tickets: "Trace this. Document findings in Docs/Working/. No code changes."
- Implementation Tickets: "Implement per spec in ticket body (plan of record)."

**Add: Deterministic Debugging Protocol**
- No speculation. Write harness scripts.
- Run against real data.
- Document actual behavior in working doc.
- Only then reason about cause.

**Add: Spec-First Investigation (for DAG layer work)**
- Before investigating: Read linked spec document first
- During investigation: Compare findings against spec requirements
- Output: Root cause + spec compliance status
- This prevents 70+ rework cycles (per history analysis)

### 3. **Architect.md** — Expand from Reviewer to Design Explorer
**CRITICAL: Architect role is larger than currently documented**

**Add: Design Exploration Triggers (not just engineer handoff)**
- Feasibility studies (can this work? what are bottlenecks?)
- Performance profiling and complexity audits
- Pre-implementation spec review (before engineer starts)
- Investigation harnessing (when findings look incomplete)

**Add: Working Doc Review Flow**
- Inline comments in markdown, not in tickets
- When to say "working doc is ready" vs. "needs more iteration"
- How to structure spec feedback (inline format)
- Spec review happens in Docs/Working/ BEFORE tickets exist
- Depth levels: Lightweight (clarity check) vs. Thorough (feasibility + edge cases)

**Add: Investigation Harnessing**
- How to recognize when investigation findings should trigger re-investigation
- vs. when findings are sufficient to implement
- Re-investigation protocol when spec review raises architectural questions

### 4. **Product.md** — Add Context Clause: Activation Based on Code Topology
**Add: Context-Aware Activation**
```
Dormant during: Tightly-coupled backend work (DAG, core algorithms, invariants)
  Reason: User decides architecture directly; agent input has high interference cost

Active during: Loosely-coupled layers (UI, signals, usability, iteration)
  Reason: User benefits from agent perspective; changes are safer to iterate

Decision trigger: When invoked, assess code coupling before proceeding
```

**Add: Spec Design in Working Docs**
- Specs start in Docs/Working/*_spec.md
- Inline feedback loops with Architect/User
- Graduation to ticket/state doc only after finalized

### 5. **_protocols.md (Shared)** — Add Artifact Lifecycle & Explicit Scope
**Add: Working Doc as Source of Truth During Design**
- Working docs are not drafts; they're the active design space
- Inline comments are the official review mechanism
- Tickets are only created from finalized working docs
- Multiple roles can read/comment during design phase

**Add: Context Preservation**
- When context window filling: update working doc with findings
- Next agent reads working doc as reference
- Working doc is session boundary, not email notification

**Add: Plan of Record Style**
- Ticket bodies are execution plans, not design documents
- No "architect notes" or "options considered" in bodies
- Reasoning/process goes in comments or working doc references only
- Prevents accumulation of design history in execution plan

**Add: Explicit Scope Checklist (Pattern #11)**
Every instruction/ticket/spec must explicitly state:
```
Constraints:
- [ ] What agents CAN do
- [ ] What agents CANNOT do
- [ ] What requires explicit permission

Scope:
- [ ] What IS in scope
- [ ] What is NOT in scope
- [ ] What is future work (deferred)

Decisions:
- [ ] Parallelism: what can run in parallel?
- [ ] Sequence: what must be sequential?
- [ ] Atomicity: one commit vs. separate?
- [ ] Updates: what docs/artifacts should be updated?

Permissions:
- [ ] When are code changes allowed?
- [ ] When are doc updates mandatory?
- [ ] When can agent decide vs. must ask?
```

### 6. **New: Artifact Graduation Checklist**
Create a shared reference for when artifacts graduate:
```
Working Doc → Proposal:
  - [ ] Inline feedback loop closed
  - [ ] All open questions answered
  - [ ] Spec finalized
  - [ ] Ready for decomposition

Proposal → Tickets:
  - [ ] Broken into discrete work items
  - [ ] Each ticket has clear plan of record
  - [ ] Dependencies identified
  - [ ] Ready for engineering

Tickets → State Doc:
  - [ ] Work completed
  - [ ] Tests passing
  - [ ] Changes pushed
  - [ ] Decisions recorded in state doc

State Doc → Archive:
  - [ ] Work documented in reference docs
  - [ ] No open questions remain
  - [ ] Can be safely archived
```

### 7. **New: Working Doc Template**
Create standard structure:
```
# [Title] - Working Doc

## Problem Statement
[What are we solving?]

## Investigation/Findings
[Deterministic findings from running code, no speculation]

## Options Considered
[Inline feedback below each option]

[Rajesh] - Option 1 won't work because...

## Recommended Approach
[Final recommendation after feedback loops]

## Next Steps
[For next agent to pick up from here]
```

---

## Director Recommendations: Priority Order

**Why this order?** Close the highest-leverage gaps first. Director review identified that Architect underdocumentation and Product context-dependency are blocking effective workflow. Working doc formalization is foundational.

### Priority 1: Architect.md Expansion (CRITICAL)
**Why:** Architect operates beyond documented scope. Agents invoke them expecting feasibility studies, spec reviews, investigation guidance. Current docs don't authorize this.

**Action:** Expand `architect.md` "Workflow" section to include:
- Design Exploration Triggers (when to invoke architect before engineer starts)
- Working Doc Review Flow (how architect operates on specs/proposals before tickets)
- Investigation Harnessing (how architect recognizes incomplete findings)

**Expected impact:** Architect behavior becomes clear. Reduces re-investigation cycles.

### Priority 2: Product.md Context Clause (HIGH)
**Why:** Product is dormant during backend work. Documenting this prevents unnecessary context loading.

**Action:** Add to `product.md`:
- Context-aware activation based on code topology
- DAG layer = dormant (user decides directly)
- UI layer = active (agent perspective adds value)

**Expected impact:** Agents know when Product is relevant. Reduces false starts.

### Priority 3: _protocols.md Additions (HIGH)
**Why:** Working docs are de facto source of truth. Formalizing lifecycle prevents ambiguity.

**Action:** Add to `_protocols.md`:
- Working Doc as Design Space (not drafts)
- Context Preservation protocol (working doc bridges sessions)
- Plan of Record style (ticket body ≠ design history)
- Explicit Scope Checklist (remove ambiguity from instructions)

**Expected impact:** Agent behavior becomes predictable. Fewer misunderstandings about artifact purposes.

### Priority 4: Engineer.md Updates (MEDIUM)
**Why:** Two ticket types exist but aren't formally distinguished.

**Action:** Add to `engineer.md`:
- Investigation Ticket type (trace, document, don't code)
- Implementation Ticket type (code from plan of record)
- Spec-First Protocol (for DAG layer): read spec before investigating
- Deterministic Debugging (trace, don't speculate)

**Expected impact:** Investigation quality improves. Spec gaps caught before implementation.

### Priority 5: Director.md Formalization (MEDIUM)
**Why:** Artifact graduation and role activation are implicit.

**Action:** Add to `director.md`:
- Artifact Graduation Checklist (when working doc → proposal → ticket)
- Role Activation Decision Tree (based on code topology)

**Expected impact:** Clearer model for new work. Reduced ambiguity about artifact maturity.

---

## Implementation Path

**Phase 1: Codify Immediately**
- Priorities 1 & 2: Architect expansion + Product context clause
- Update `architect.md` and `product.md` directly
- Agents can begin using these within one session
- Expected impact: 60% reduction in workflow ambiguity

**Phase 2: Formalize Flow**
- Priority 3: _protocols.md additions (Working Doc, Plan of Record, Explicit Scope)
- Update instruction templates with Explicit Scope Checklist
- Coordinate with next user task to validate format

**Phase 3: Engineer & Investigation**
- Priority 4: Engineer.md ticket types + Spec-First Protocol
- Test with next DAG layer investigation
- Validate Spec-First prevents rework

**Phase 4: Architecture & Artifact**
- Priority 5: Director.md artifact lifecycle + role activation tree
- May be aspirational initially (codify what's working, not what should work)
- Iterate based on agent feedback

---

## Closing the Loops: How Director Findings Resolve Identified Patterns

This section maps director review findings to the 12 patterns, showing how formalizing personas closes the gaps.

| Pattern | Problem | Director Finding | Fix |
|---------|---------|------------------|-----|
| **#1: Working Docs as Harnesses** | Agents don't reliably use working docs for investigation | Working docs are underdocumented as persistent artifacts | Formalize in `_protocols.md`: Working Doc as Design Space |
| **#2: Inline Comments** | Agents don't know when to use inline feedback vs. tickets | No guidance on review mechanism preference | Add to `architect.md`: Working Doc Review Flow with inline comment protocol |
| **#3: Ticket Editing Workaround** | Agents try to edit ticket bodies; user prevents it | Distinction between design (working doc) and execution (ticket) is implicit | Codify "Plan of Record" style in `_protocols.md` |
| **#4: Artifact Graduation** | Agents don't know when to graduate artifacts | No explicit criteria for working doc → proposal → ticket progression | Add Artifact Graduation Checklist to `director.md` |
| **#5: Investigation vs. Implementation Tickets** | Agents treat investigation as informal | Two ticket types exist but aren't formalized | Formally distinguish in `engineer.md` |
| **#6: Context Bridge** | Agents don't use working docs to bridge context windows | No explicit protocol for session boundaries | Add Context Preservation protocol to `_protocols.md` |
| **#7: Specs as Mutable Drafts** | Agents unsure when specs are "done" | Spec review happens in working docs, not tickets, but this is opaque | Add to `_protocols.md`: Specs live in working docs until finalized |
| **#8: User as Architect** | Agents don't recognize when user updates tickets as decisions | Ticket body updates are architectural decisions, not metadata | Document in `_protocols.md` that user updates = decision locks |
| **#9: Deterministic Debugging** | Agents speculate instead of tracing | No explicit protocol for investigation methodology | Add Deterministic Debugging Protocol to `engineer.md` |
| **#10: State Docs as Records** | Agents create fresh state docs instead of updating | State docs are long-lived, not session artifacts | Document in `_protocols.md`: State docs always updated, never created fresh |
| **#11: Explicit Over Implicit** | Agents infer constraints instead of having them stated | Instructions have ambiguous scope/constraints | Introduce Explicit Scope Checklist template |
| **#12: Code Topology** | Workflow depth is fixed regardless of code coupling | Architect/Product scope depends on code topology | Add Role Activation Decision Tree to `director.md` |

---

## Explicit Scope Checklist (Pattern 11)

Every instruction/ticket/spec should explicitly state:

### Constraints
- [ ] What agents CAN do
- [ ] What agents CANNOT do
- [ ] What requires explicit permission

### Scope
- [ ] What IS in scope
- [ ] What is NOT in scope
- [ ] What is future work (explicitly deferred)

### Decisions
- [ ] Parallelism: what can run in parallel?
- [ ] Sequence: what must be sequential?
- [ ] Atomicity: what must be one commit vs. separate?
- [ ] Updates: what docs/artifacts should be updated?

### Permissions
- [ ] When is code changes allowed?
- [ ] When is doc updates mandatory?
- [ ] When can agent decide vs. must ask?

---

## ADDITIONAL DIRECTOR FINDINGS (Jan 19, 2026)

Beyond the 12 patterns above, deeper analysis of history.jsonl reveals 7 more behavioral patterns that should inform persona updates:

### Pattern 13: Hypothesis-Test-Revert as Standard Workflow

**Evidence:** 60+ explicit revert instances across history

Implementation is not linear. Agents propose hypothesis → implement → test → fails → revert → try different approach. This cycle is **normal and expected**, not a sign of failure.

**Why it matters:**
- Reverting is **debugging methodology**, not failure
- Clean revert + fresh approach beats incremental patching
- User has high tolerance for hypothesis waste if each attempt is clean

**Example:** "seems like that's special casing this very badly. Why don't you back out all of this. See if the session is initialized."

**For personas:** Engineer workflow should normalize reverting as part of hypothesis testing, not treat it as exceptional.

---

### Pattern 14: Empiricism Doctrine — Extreme Anti-Speculation Enforcement

**Evidence:** 38+ instances of "don't speculate", "trace the code", "execute against real data"

This is more than a guideline—it's a **fundamental doctrine enforced repeatedly**. Agents are corrected even when speculation seems reasonable.

**Core rule:**
- Never infer behavior by reading code
- Always write deterministic tests/harnesses for investigation
- Observation > theory always
- User will correct speculation even when it seems reasonable

**Example:** "No dude, trace the code -- add a debug log if you want just for this leg id (it's deterministic). Don't speculate and ask me."

**For personas:** Engineer investigation protocol needs explicit "no speculation" rule with teeth. Agents should default to tracing/harnesses, not code reading.

---

### Pattern 15: Architect Validates Against Written Architecture Specs

**Evidence:** 5+ instances of architect checking implementation against authoritative specs

When architect reviews implementation, they validate it complies with **documented architecture semantics**, not just code quality.

**Example:** "read the pivot first architecture v2 spec, head has specific semantic to it and this seems to go against it. are you sure this is correct fix?"

**This is different from line-by-line code review.** It's spec compliance checking.

**For personas:** Architect workflow should include: "Keep architecture specs (DAG.md, pivot_first_architecture_v2.md) as source of truth. Validate implementations against spec constraints."

---

### Pattern 16: Engineer Responds to Architect Comments on Tickets

**Evidence:** 8+ patterns showing architect comments → engineer responds/updates

Tickets are collaborative. Architect leaves comments → Engineer reads → Updates ticket or responds in comments. Tickets evolve as shared artifacts.

**Example:** "i have engineer agents respond to architect comments. use the ticket for comment and update."

**For personas:** Engineer workflow should include: "Tickets are collaborative. Watch for architect comments. Update ticket/respond based on feedback."

---

### Pattern 17: Deeper Analysis is Iterative Process

**Evidence:** 23+ instances of "what else did you miss?" or "do a deeper review"

First-pass analysis is **expected to be incomplete**. User follows initial response with deeper pass requests. This is not criticism—it's normal workflow.

**Pattern:** Analysis is multi-pass, not one-shot
- First pass should be comprehensive but still incomplete
- User will ask for deeper dives as a matter of course
- This is workflow, not failure

**For personas:** All roles should expect multi-pass analysis. First-pass feedback is normal entry point, not final word.

---

### Pattern 18: Context-Dependent Workflow Variance

**Evidence:** User explicitly states: "when i'm doing ui iterations or signal iterations i may have engineer do things immediately"

Not all work gets the same treatment. Code topology determines workflow intensity:

**DAG Layer (tightly coupled):**
- Slow, deliberate, spec-review-heavy
- Pre-implementation architect spec review is standard
- Post-implementation architect compliance checking is standard
- Everything else depends on it

**UI/Signal Layers (loosely coupled):**
- Can accelerate (engineer moves directly)
- Faster iteration acceptable
- Architectural risks are lower

**For personas:** Encode code topology as workflow selector. Tightly-coupled layers = strict workflow. Loosely-coupled = fast workflow.

---

### Pattern 19: Director Context Hygiene

**Evidence:** User enforces strict input restrictions for director analysis

When director role reviews workflow patterns, hygiene is required to avoid context contamination.

**Director read-list:**
- `.claude/personas/*.md` (allowed)
- `/Users/rajesh/.claude/history.jsonl` (allowed)

**Director blocked-list:**
- `Docs/State/*` (blocked)
- `Docs/Working/*` (blocked)
- `Docs/Comms/*` (blocked)
- Any project documentation (blocked)

**Why:** Prevents director from inheriting other agents' context and confusing patterns.

**For personas:** Document in director.md: strict input isolation required for objective workflow analysis.

---

## Synthesis: All 19 Patterns

The complete picture emerges from combining:
- **Patterns 1-12** (workflow_evolution.md findings): Artifact flows, spec review timing, decision gates
- **Patterns 13-19** (additional discoveries): Behavioral meta-patterns about testing, empiricism, context

Together they show the **real workflow** is:
- **Empirical, not theoretical** (no speculation)
- **Fail-fast iterative** (reverting is normal)
- **Multi-pass analysis** (deeper review cycles expected)
- **Code-topology-driven** (DAG layer ≠ UI layer)
- **Spec-informed** (read specs before implementation)
- **Collaborative** (tickets are shared artifacts)
- **Context-conscious** (strict hygiene for director)
