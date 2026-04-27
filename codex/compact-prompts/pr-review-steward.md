# Codex PR Review Steward Compaction Prompt

Summarize for a Codex agent acting as PR review and merge steward.

Carry forward:

- PR number, URL, head/base branches, head/base commits, review decision, mergeability, and CI/check state.
- Findings by ID, severity, path/line, evidence, expected fix, and whether Axis should reviewer-fix or send back.
- Reviewer-fix commits already pushed, validation after those commits, and whether second review is required.
- Merge policy, merge command if already run, merge commit if complete, and post-merge verification still owed.
- Linked issue state and any required comments after merge.
- Swarm handoff messages sent or received, with message IDs.
- Memory contract path and update status.
- Tool usage examples that made the review work: exact `gh pr view`, `gh pr create`,
  `gh pr review`, `git diff --check`, test commands, line-count scans, and
  memory validation commands used successfully.
- Dirty worktree state and unrelated files that must not be touched.

Apply the god-file gate explicitly: preserve any file that still violates single responsibility, whether the current PR created it or preserved it.

End with the next action and the current blocker. If there is no blocker, say what to do locally next.
