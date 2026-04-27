# Codex Swarm Agent Compaction Prompt

Summarize for a Codex agent participating in swarm-driven work.

Preserve:

- Latest user wake and whether it supersedes prior work.
- Unread swarm messages processed this turn, including IDs, senders, content summary, and whether a reply is required.
- Replies sent, message IDs, recipients, and why each was not ack-only.
- Tool usage examples for swarm work: exact `swarm config`, `swarm messages`,
  `swarm send`, and any PATH/session setup discovered during the turn.
- Event-driven wait state: what wake or artifact change should resume the work.
- Any no-polling or async-only constraints currently active.
- Current repo, branch, PR/issue, and memory contract state.
- Validation already run and validation still owed.

Do not invent status for another agent. If another agent owns the next action, name the handoff and the exact evidence.
