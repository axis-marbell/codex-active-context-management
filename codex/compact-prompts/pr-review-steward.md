# Codex PR Review Steward Compaction Prompt

Summarize for a Codex agent acting as PR review and merge steward.

Carry forward:

- PR number, URL, head/base branches, head/base commits, review decision, mergeability, and CI/check state.
- Findings by ID, severity, path/line, evidence, expected fix, and whether Axis should reviewer-fix or send back.
- Reviewer-fix commits already pushed, validation after those commits, and whether second review is required.
- Merge policy, merge command if already run, merge commit if complete, and post-merge verification still owed.
- Linked issue state and any required comments after merge.
- Swarm wake and handoff messages sent or received, with message IDs, senders,
  content summary, and whether a reply is required.
- Replies sent, with message IDs, recipients, and why each reply was not
  ack-only.
- Memory contract path and update status.
- Tool usage examples that made the review work: exact `gh pr view`, `gh pr create`,
  `gh pr review`, `git diff --check`, test commands, line-count scans, and
  memory validation commands used successfully. Preserve swarm command examples
  too when swarm affected the PR: `swarm config`, `swarm messages`, `swarm send`,
  and any PATH/session setup discovered.
- Dirty worktree state and unrelated files that must not be touched.
- Event-driven wait state: what wake, review update, CI result, or artifact
  change should resume the work.

Apply the god-file gate explicitly: preserve any file that still violates single responsibility, whether the current PR created it or preserved it.

End with the next action and the current blocker. If there is no blocker, say what to do locally next.
Do not invent status for another agent. If another agent owns the next action,
name the handoff and the exact evidence.
