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
three agents in parallel is the practical ceiling, and they should be decoupled well enough that
coordination is a single message when a PR lands.

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
   the report is a symptom, find the cause before proposing a fix.
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
5. **Build, install, and restart the thing** in its real location, then tell him it is ready to
   test. The specifics are in each repo's own `AGENTS.md`.
6. **Wait for his verification.** He tests it himself — physically, in the app, on the device.
   Iterate on his feedback until he signs off. He will waive this step explicitly when there is
   nothing to check or he is unavailable; do not assume the waiver.
7. **Open the PR** once he has signed off.
8. **Run the review loop** below until it exits. **A review fix is new code.** It lands after the
   suite ran in step 4 and after he signed off in step 6, so neither covers it: re-run the suite
   before merging, and if the fix changes behaviour he verified, say so and get it re-checked
   rather than treating the earlier signoff as still valid. Merging on an unverified repair can
   ship a defect worse than the one it fixed.
9. **Squash merge**, then delete the branch and any worktree.
10. **Rebuild and restart** if anything changed since step 5, clean up stale builds and binaries,
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
2. **Correctness only.** No document nits, no wording preferences, no process commentary.
3. **Any unresolved P1 blocks.** A round that returns only P2 or lower, or a clean review, exits
   the loop. Do not keep chasing P2s and P3s.
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

## 6. Conventions

- **Never commit directly to the default branch.** Work on a branch, merge through a PR.
- **Rajesh reviews in the PR.** Do not merge something he is still reading.
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
