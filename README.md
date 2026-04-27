# Codex Active Context Management

External context monitoring and compaction prompts for Codex CLI agents.

## What CACM Does

Codex Active Context Management watches Codex session JSONL from outside the
agent process and sends tmux reminders when context gets large or important
workflow milestones appear. It does not compact automatically and it does not
replace Codex config. It gives the agent timely, sourced state so the agent can
choose the right moment to compact, file memory, or keep working.

CACM is adapted from `finml-sage/active-context-protocol` for Codex CLI:

- monitors Codex session logs under `~/.codex/sessions`
- reads Codex `event_msg` entries where `payload.type == "token_count"`
- uses `payload.info.total_token_usage.total_tokens` as current context usage
- delivers reminders through tmux using the existing two-call send-keys pattern
- ships file-backed Codex compaction prompts under `codex/compact-prompts/`

## Features

- Token monitoring with configurable threshold, default `180000`
- Compaction reminders with grace period, cooldown, and stacking prevention
- Memory filing reminders for milestones such as PR merges, issue closures,
  commits, and PR creation
- tmux delivery for reminders into the active Codex session
- Session tracking for `rollout-*.jsonl` files under `~/.codex/sessions`
- Incremental JSONL reads from the last known byte position
- YAML configuration with a zero-dependency fallback parser
- CLI commands: `cacm init`, `cacm start`, `cacm status`, `cacm config`,
  `cacm stop`
- Codex `config.toml` example for `experimental_compact_prompt_file`,
  `compact_prompt`, and `model_auto_compact_token_limit`

## Requirements

- Python 3.11+
- tmux for reminder delivery
- Codex CLI
- No required runtime Python dependencies

PyYAML is optional. Without it, CACM uses the built-in minimal YAML parser for
the default config shape.

## Quick Start

```bash
pip install -e .

cacm init
# writes ~/.codex-active-context-management/config.yaml

# Edit tmux_session to the tmux session where Codex is running.
cacm config
cacm start
cacm status
cacm stop
```

To use the bundled Codex compaction prompt, copy or reference the prompt path in
`~/.codex/config.toml`:

```toml
experimental_compact_prompt_file = "~/.codex/compact-prompts/active-context.md"
model_auto_compact_token_limit = 180000

# For short experiments only:
# compact_prompt = "Summarize current work into durable state, preserving blockers, branches, commits, validation, and next actions."
```

The repo includes a fuller example at `codex/config.toml.example`.

## Configuration

CACM reads from `~/.codex-active-context-management/config.yaml`. Run
`cacm init` to generate the default:

```yaml
token_threshold: 180000
polling_interval: 30

warmdown_interval: 120
grace_period: 300
tmux_session: ""
log_file: "~/.codex-active-context-management/cacm.log"

idle_threshold: 5.0

codex_sessions_dir: "~/.codex/sessions"
codex_config_path: "~/.codex/config.toml"
compact_prompt_file: "~/.codex/compact-prompts/active-context.md"

compaction:
  enabled: true
  threshold: 180000
  cooldown: 120

memory_filing:
  enabled: true
  grace_after_event: 60
  patterns: []
```

The only required delivery setting is `tmux_session`.

## Compaction Prompts

CACM ships three prompt templates:

- `codex/compact-prompts/active-context.md` for general Codex work
- `codex/compact-prompts/pr-review-steward.md` for PR review and merge work
- `codex/compact-prompts/swarm-agent.md` for swarm-driven work

The prompts preserve operational state that often gets lost during compaction:
current user corrections, repo and branch state, PR numbers, swarm message IDs,
commands run, validation status, memory paths, next action, and concrete tool
usage examples such as working CLI flags or config keys. This is intentional:
after compaction, the next Codex turn should not need to rediscover basic tool
usage that was already learned in the session.

## Architecture

```text
Codex CLI agent in tmux
    |
    | writes
    v
~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl
    ^
    | incremental read
    |
CACM monitor
    |
    +-- FileTracker ---------> finds latest rollout JSONL and detects rotation
    +-- TokenMonitor --------> reads event_msg/token_count entries
    +-- CompactionTrigger ---> threshold, grace, cooldown, pending checks
    +-- MemoryFilingTrigger -> detects milestone patterns
    +-- DeliverySystem ------> sends reminders through tmux
```

Token monitoring uses the latest Codex token-count event:

```json
{
  "type": "event_msg",
  "payload": {
    "type": "token_count",
    "info": {
      "total_token_usage": {
        "input_tokens": 1234,
        "cached_input_tokens": 100000,
        "output_tokens": 500,
        "total_tokens": 180500
      }
    }
  }
}
```

Legacy assistant `message.usage` entries are still parsed as a compatibility
fallback.

## What the Agent Sees

Compaction reminder:

```text
[CACM] Codex context at 180500 tokens (100% of 180000 threshold). Consider compacting with the configured Codex compaction prompt when ready.
```

Memory filing reminder:

```text
[CACM] Significant work detected. Consider filing to memory: takeaways, decisions, facts, or proverbs.
```

## Development

```bash
pip install -e ".[test]"
python3 -m pytest tests/ -v
```

The codebase keeps zero required runtime dependencies by design.

## License

MIT
