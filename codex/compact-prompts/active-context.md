# Codex Active Context Compaction Prompt

Summarize the live session so the next Codex turn can continue without guessing.

Preserve concrete, sourced state:

- User's latest instruction and any corrections that override earlier instructions.
- Active repo, branch, base commit, dirty files, untracked files, and worktree warnings.
- Open PRs/issues being handled, review state, merge blockers, and who owns the next action.
- Swarm wake messages, message IDs, senders, decisions that changed the
  workflow, and whether any reply is required.
- Replies sent, with message IDs, recipients, and why each reply was not
  ack-only.
- Commands already run and their material outcomes, including failed commands.
- Tests/checks already run, exact pass/fail status, and gaps that remain.
- Tool usage examples learned this session: commands, flags, config keys,
  API/CLI shapes, and working invocation patterns that prevented rediscovery.
- Files changed intentionally, files dirty before the work, and files that must not be reverted.
- Memory contracts/templates updated or still needing updates.
- Secrets/config paths mentioned by the user, without revealing secret values.
- Event-driven wait state: what wake or artifact change should resume the work.
- Next concrete action, with the blocker if one exists.

Do not compress away:

- Review findings and severity.
- Commit hashes, PR numbers, issue numbers, branch names, and message IDs.
- User policy corrections such as no blocking polling, no ack-only swarm replies, or god-file review gates.
- Concrete tool examples that are still relevant after compaction, such as
  `swarm messages --status unread --limit 20 --json`, `gh pr view ... --json`,
  `memory validate <path>`, or Codex config keys like `experimental_compact_prompt_file`.
- Anything that distinguishes "waiting on another agent" from "continue locally now."

Do not invent status for another agent. If another agent owns the next action,
name the handoff and the exact evidence.

Keep the summary operational. Prefer terse bullets with file paths and IDs over narrative.
