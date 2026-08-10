# Working in Rajesh's supporting repos

This file is the standing contract for the small repos that support the main work: **deskbar**,
**office-automate**, **finviz**, and **session-manager**. Read all of it. There is no second rules
file to chase, and there are no persona documents.

Each repo's own `AGENTS.md` covers what is specific to it — what it builds, how to run it, where
its config lives. This file covers how work runs everywhere.

## The shape of the work here

A ticket is small. One agent owns it end to end: implement, get it verified, open the PR, drive
the review, merge, clean up. Rajesh usually files it because something broke or he wants a
feature, and he is often waiting on the result.

That means no orchestration. No waves, lanes, gates, or handoff protocol. If you find yourself
designing a multi-agent plan for one of these repos, you have misread the size of the job. Two or
three agents in parallel is the practical ceiling, and their work must be decoupled well enough
that coordination is a single message when a PR lands.

**Parallel agents never share a checkout.** Each works in its own worktree on its own branch.
Picking non-overlapping tickets is not enough: agents in one checkout share the index, HEAD, and
the current branch, so one agent's checkout or commit silently absorbs or discards another's work
regardless of how unrelated the tickets are. If a worktree is not an option, serialize — one agent
touching the repo at a time.

---

## 1. Talking to Rajesh

His attention is the scarce resource. Assume he was looking at something else a minute ago and
does not have your context loaded.

- **Define every term before you use it.** Never assume a name from an earlier message survived.
  He read that message; he did not memorise it.
- **State the conclusion first, then the reasoning.** Active sentences, plain English.
- **Give him decisions, not questions.** Come with a recommendation and a reason. "Should we
  delete this?" is not a question for him; "I recommend deleting this because X — confirm?" is.
  Sleuth first: if the answer is discoverable, discover it.
- **Answer what he actually asked.** If he asks yes or no, lead with yes or no.
- **When a document is the deliverable**, it is a memo plus appendices. The memo is his entire
  decision surface and is capped at what he will actually read — six pages printed. Appendices are
  unlimited and must be detailed enough that an implementing agent cannot come back with a
  blocking question. Include a chart, diagram, or table wherever one would beat a paragraph.

---

## 2. The work loop

1. **Understand the actual problem.** These tickets usually come from something he observed. If
   the report is a symptom, find the cause before proposing a fix. **Do not diagnose by reading
   code alone** — run it against real data, add temporary tracing, and compare what happens to
   what should happen. A plausible theory that was never executed is not a diagnosis, and this
   project has a long history of confident wrong ones.
2. **Smoke test early.** Confirm you can read the state, reach the device, or reproduce the bug
   *before* writing the full implementation. Writing the whole change and then discovering the
   basic path does not work is the most common way to waste a cycle here.
3. **Implement**, matching existing patterns in the repo.
4. **Write tests and run the suite.** New behaviour and bug fixes get test coverage, and the
   repo's test command runs green before anything is deployed or reviewed. His smoke test exercises
   the path he is thinking about; it cannot catch a regression somewhere else, so it is not a
   substitute for the suite. If you hit a failure that looks pre-existing or unrelated, do not
   silently skip it — file an issue with the test name, the assertion, and whether it predates your
   change, and reference it in your PR.

   Run the repo's **build verification too** — type check, lint, whatever it has. A green test run
   with a red lint still fails CI after he has signed off. For a bug fix, write the test so it
   **fails before your change and passes after**; a test written afterwards can pass for the wrong
   reason and cover nothing.

   **The green must be on the exact code you ship.** Any edit after a green run invalidates it —
   his feedback in step 6, a review fix in step 8, a rebase, a last-minute tidy. Run the suite
   again before you move on. This applies every time round the loop, not only the first.
5. **Build, install, and restart the thing** in its real location, then tell him it is ready to
   test. The specifics are in each repo's own `AGENTS.md`.
6. **Wait for his verification.** He tests it himself — physically, in the app, on the device.
   Iterate on his feedback until he signs off, re-running step 4 and redeploying on each pass —
   his feedback produces code, and code he has not seen running is not code he has verified. He
   will waive this step explicitly when there is nothing to check or he is unavailable; do not
   assume the waiver.
7. **Open the PR** once he has signed off, on a green run of the code you are actually shipping.
   Before you do: revert any debug prints or tracing you added in step 1, and check that anything
   you built is actually wired in — a component that exists but is never imported, a handler never
   registered, or a config option nothing reads is not a finished ticket. Reference the issue with
   `Fixes #N`, exactly once, and confirm the number is the issue this change actually fixes.
8. **Run the review loop** below until it exits. A review fix is new code and lands after both
   gates: re-run step 4 before merging, and if the fix changes behaviour he verified, say so and
   get it re-checked rather than treating the earlier signoff as still valid. Merging on an
   unverified repair can ship a defect worse than the one it fixed.
9. **Squash merge**, then delete the branch and any worktree. If the merge had conflicts, check
   afterwards that features merged since your branch point still exist — conflict resolution can
   silently drop code that was never in your diff, so neither review nor his smoke test would
   catch it.
10. **Return the persistent checkout to the default branch and pull.** Merging from a worktree
    leaves it sitting at its pre-merge commit, so the next ticket branches from code that is
    missing the change you just landed.
11. **Rebuild and restart** if anything changed since step 5, clean up stale builds and binaries,
    and report the repo is clean.

---

## 3. The review loop

Request a review with `sm request-codex-review <pr-number>`. Treat the response as registration
only, then go idle — do not poll. If Session Manager cannot take the request, post `@codex review`
as a PR comment, check back after five minutes, again after five more, and re-post if nothing has
landed after ten.

Before acting on a review, confirm it belongs to your current request and was posted after your
latest push. A review existing is not enough on its own.

Then:

1. **Classify every finding: valid, partially valid, or invalid.** Do not skip this. A review is
   not gospel — push back with reasoning where it is wrong.
2. **Correctness only** — no document nits, no wording preferences, no nits about following
   process for process's sake. That excludes process *preference*, not process *correctness*:
   when the thing under review is a workflow, an instruction file, CI, or a deploy step, its
   behaviour is the correctness surface, and a defect in it is a correctness finding however
   procedural it sounds. "This deploys without re-running the tests" is a bug, not a nit.
3. **Any unresolved P1 blocks.** A P1 is resolved either by fixing it or by answering it: a P1 you
   classify invalid, with your reasoning posted on the PR, is resolved and does not block. It is
   unresolved only while it is neither fixed nor answered — otherwise a single false positive
   strands the PR forever, since there is no code change to push for the next round. If the
   reviewer re-raises the same P1 after reading your reasoning, that is a real disagreement:
   escalate it rather than looping.
   A round that returns only P2 or lower, or a clean review, exits the loop. Do not keep chasing
   P2s and P3s.
4. Fix, push, and re-review at the exact head. Fewest rounds to correctness — which does not mean
   dropping correctness issues.

---

## 4. Finishing

Before you report done or ask to be retired, check all of it:

- Worktrees deleted. Local **and** remote branches deleted — a squash merge does not reliably
  delete the remote branch, so verify with `git ls-remote --heads origin <branch>`.
- Old builds and binary detritus cleaned up; the service rebuilt and running.
- Tickets closed, or named explicitly if you left one open on purpose.
- No orphaned processes or listeners from your work.

Keep the open-ticket count low — around four or five. Anything that does not block progress or
yield a real improvement gets closed rather than carried.

Put anything worth keeping where it can be found later: a PR comment, a doc in the repo, a ticket.
Not in your context, which retires with you.

---

## 5. Session Manager is different in one way

`sm` is the infrastructure the main project's workflow runs on. If it breaks, agents everywhere
stall, and Rajesh may be asleep. It therefore has a **named maintainer** — a standing seat that
takes most tickets and is expected to be reachable when he is not.

When something is broken:

1. **Unblock the workflow first.** Get agents moving again with the smallest safe intervention.
2. **Then find the real cause.** The unblock is not the fix, and stopping there is how the same
   failure returns tomorrow.
3. **Then fix it properly** and run the normal loop above.

Say clearly which of the three you have done. "Unblocked, root cause found, fix in review" is a
useful status; "fixed" when you have only unblocked is not.

**If you hit an `sm` bug while working in any other repo, report it — do not just work around it
and move on.** File it in `rajeshgoli/session-manager`, then reach the maintainer directly:

```bash
sm roster                     # authoritative: shows which roles are actually seated
sm lookup <role>              # resolves that role to a session id
sm send <session-id> 'short description + the issue link'
```

Read the roster first and use the role name it shows. Role keys drift — stale registrations can
resolve to a session that is no longer the seated maintainer, so a lookup that returns *something*
is not proof you reached the right seat. A workaround that stays in your context is a bug nobody
else knows about, and the next agent hits it too.

Other things specific to this repo: the server is Rust, and the Python CLI is an older
implementation — do not treat it as current. Restart with `launchctl`. Control surfaces belong in
the Android app first and the web app second: putting a control only in the CLI defeats the
purpose, because the CLI is exactly what is unreachable when he is away from his machine.

---

## 6. Working with Session Manager

- **`sm register <role>`** takes a named seat; `sm lookup <role>` resolves it and `sm roster` lists
  what is actually seated. A standing seat that never registers cannot be found by anyone.
- **Report completion to whoever dispatched you.** If an agent sent you the work, `sm send` your
  result back to it when you finish or get blocked. It is idle waiting for you and nothing else
  will wake it. If Rajesh dispatched you, finishing in your own console is enough.
- **Waiting on another agent**: go idle, do not poll. If you need a status, `sm what <id>`
  summarises it without pulling their output into your context — use it sparingly. If an expected
  reply never comes, `sm wait <id> <timeout>` (600s is reasonable for a review) and then message
  them directly rather than waiting forever.
- **Waiting on something external** — a build, a log, an exit-code file — use `sm watch-job add`.
  Never `sleep`, `tail -f`, or a polling loop.
- **On `[sm remind]`**: run `sm status "what you are doing now"` and carry on. It is not an
  interrupt. Call `sm task-complete` when you finish so reminders stop firing at an idle seat.
- **`sm context-monitor enable`** warns at 50% and flags critical at 65%. This matters for Claude
  seats, where a single agent owning a ticket end to end can compact mid-loop and lose the
  ticket's history with no warning. Codex seats can let autocompact do its thing.
- **`sm handoff` erases your context** and wakes you with the doc you wrote. Write the doc first.
  Running it before writing leaves nothing to resume from.
- **File writes take workspace locks automatically.** If an edit blocks, another agent holds the
  lock — check with `sm others` rather than retrying or working around it.
- **Stop and ask when you are stuck.** Tests failing for reasons you cannot explain, the same
  failure two or three times running, or no clear way forward means escalate — not another lap.
  Assume he is at the terminal: say it there first, and set a 30-minute reminder
  (`sm remind 1800 "<what you are blocked on>"`). If he replies, cancel it. If the 30 minutes
  elapse with no reply, `sm email rajesh` and say in the subject that it blocks.

---

## 7. Conventions

- **Never commit directly to the default branch.** Work on a branch, merge through a PR.
- **Doc PRs wait for his review; code PRs do not.** A code PR follows the loop above and merges on
  convergence — do not hold it for him unless he has explicitly said he is reading it.
- **Agents talk to each other with `sm send`.** Claude agents default to the wrong transport and
  must be told. Never poll another agent's output — go idle and you will be woken. Keep
  inter-agent traffic to what the work requires; no status chatter.
- **Never wrap a message containing backticks or `$()` in double quotes.** The shell substitutes
  before Session Manager ever sees it, which can run a local command and corrupt the message. Use
  single quotes, a heredoc, or a file payload. This bites hardest on `sm send` when you are
  relaying review text or code, but it is a general shell rule.
- **Email him only for a gate that genuinely blocks on him**, or if you have not heard back in a
  long time. Say in the subject that it is blocking.
- **No AI attribution** in commits, PRs, or documents. No "Generated by", no co-author trailers.
- **Secrets live in config, not source.** A key in a source file is a security ticket.
- **Do not silently ignore an unrelated test failure.** File an issue and reference it in your PR.
- **A ticket or handoff must stand alone.** Write it so an agent with no context can pick it up
  cold. If your successor asks a question you already asked, or repeats a mistake already
  corrected, the handoff failed.
- **Check for an existing issue before filing a new one.** Duplicates are how a backlog grows
  while everyone is closing things.
- **Edit a ticket rather than commenting on it**, until work has actually started. A ticket nobody
  has worked should read as its current intent, not as a thread to reconstruct.
- **File the ticket first, then name the doc after it** — `<ticket#>_<short_name>` in the repo's
  docs directory — so the doc can be found from the ticket and back.
- **One file per concern, overwritten in place.** No date suffixes, no `_v2`, no parallel copies.
  A reader must never have to work out which of two files is live.
- **When you delete code, delete it.** No comment explaining what used to be there, no dead config
  option that nothing reads, no field left always-null. The next agent will code against it.
