# PR Review Process

Use this process whenever a pull request needs Codex review and merge handling.

## Review Loop

1. Post `@codex review` as a PR comment.
2. Wait about 5 minutes.
3. Poll the PR for a Codex review.
4. If no review was posted, wait 5 more minutes.
5. Poll again.
6. If there is still no review after 10 minutes total, post another `@codex review` comment.
7. Repeat the cycle until a review is posted.

## Review Triage

When Codex review lands:

1. Collect all review feedback.
2. Categorize each item by severity.
3. Decide whether you agree with each item.

## Exit Criteria

- If there are any important feedback items (`P1`), exit criteria are **not met**.
- If there are no important feedback items (`P2` or lower only), exit criteria are **met**.
- If the review is clean, exit criteria are **met**.

## After Feedback

1. Fix any feedback you choose to address.
2. Commit the changes.

If exit criteria are not met:

1. Push the fixes.
2. Repeat the review loop until exit criteria are met.

If exit criteria are met:

1. Merge the PR.
2. Delete the branch.
3. Delete the worktree if one was created for the branch.
4. Clean up local state and return to the appropriate base branch.

## Notes

- Do not treat “a review exists” as sufficient by itself.
- The blocking threshold is whether unresolved feedback contains any `P1` items.
- `P2` or lower feedback can still be worth fixing before merge, but it does not block exit criteria.
