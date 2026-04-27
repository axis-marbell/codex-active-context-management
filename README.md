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

- Token monitoring with configurable threshold, default `200000`
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

## Install

```bash
git clone https://github.com/axis-marbell/codex-active-context-management.git
cd codex-active-context-management

python3 -m venv .venv
.venv/bin/python -m pip install -e .
.venv/bin/cacm --help
```

The editable install keeps the prompt templates available from the cloned repo
and installs the `cacm` command into `.venv/bin/`. If you prefer activating the
virtual environment, run `source .venv/bin/activate` and use `cacm` directly.

## Setup

Create the CACM runtime config:

```bash
.venv/bin/cacm init
# writes ~/.codex-active-context-management/config.yaml
```

Find the tmux session where Codex is running:

```bash
tmux list-sessions
```

Edit `~/.codex-active-context-management/config.yaml` and set
`tmux_session` to that session name:

```yaml
tmux_session: "codex-0"
token_threshold: 200000

compaction:
  enabled: true
  threshold: 200000
```

Install the bundled compaction prompts where Codex can read them:

```bash
mkdir -p ~/.codex/compact-prompts
cp codex/compact-prompts/*.md ~/.codex/compact-prompts/
```

Add the active-context prompt to `~/.codex/config.toml`:

```toml
experimental_compact_prompt_file = "~/.codex/compact-prompts/active-context.md"
model_auto_compact_token_limit = 200000
```

The repo includes a fuller Codex config fragment at `codex/config.toml.example`.

## Run

Start the monitor in a terminal that can reach the target tmux server:

```bash
.venv/bin/cacm config
.venv/bin/cacm start
```

Check status or stop it from another shell:

```bash
.venv/bin/cacm status
.venv/bin/cacm stop
```

`cacm start` runs in the foreground. Use tmux, your service manager, or a
supervised background process if you want it to stay running after the shell
closes. The monitor writes its PID to
`~/.codex-active-context-management/cacm.pid` and delivery logs to the configured
`log_file`.

## Configuration

CACM reads from `~/.codex-active-context-management/config.yaml`. Run
`cacm init` to generate the default:

```yaml
token_threshold: 200000
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
  threshold: 200000
  cooldown: 120

memory_filing:
  enabled: true
  grace_after_event: 60
  patterns: []
```

The only required delivery setting is `tmux_session`.

## Compaction Prompts

CACM ships two prompt templates:

- `codex/compact-prompts/active-context.md` for general Codex work
- `codex/compact-prompts/pr-review-steward.md` for PR review and merge work

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
        "total_tokens": 200500
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
[CACM] Codex context at 200500 tokens (100% of 200000 threshold). Consider compacting with the configured Codex compaction prompt when ready.
```

Memory filing reminder:

```text
[CACM] Significant work detected. Consider filing to memory: takeaways, decisions, facts, or proverbs.
```

## Development

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -e ".[test]"
.venv/bin/python -m pytest tests/ -v
```

The codebase keeps zero required runtime dependencies by design.

## License

MIT
