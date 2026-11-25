# Mission Control

[![CI](https://github.com/Falplp/mission-control-rs/actions/workflows/ci.yml/badge.svg)](https://github.com/Falplp/mission-control-rs/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**Mission Control is a terminal dashboard for supervising AI-agent pipelines.** It is
built with Rust and [Ratatui](https://ratatui.rs/) and combines live agent status,
LLM output, token and cost metrics, artifacts, errors, task queues, and human
approval in one TUI.

> Prefer Czech? Read [README.cs.md](README.cs.md).

## Highlights

- Configurable multi-agent pipelines with human-in-the-loop approval
- Offline demo mode plus Ollama, OpenAI, and Anthropic providers
- Live LLM output, tool activity, metrics, errors, history, and artifact diffs
- FIFO task queue and OpenClaw Gateway WebSocket integration
- SQLite persistence and a live JSONL event adapter
- Sandboxed workspace access with path validation and a command allowlist
- Webhook and Telegram notifications
- Keyboard and mouse navigation, multiple themes, and Docker support

## Requirements

- A terminal with UTF-8 and color support
- [Rust](https://www.rust-lang.org/tools/install) with the 2024 edition supported
- Optional: Docker, Ollama, or credentials for a cloud LLM provider

## Quick start

Clone the repository and start the application:

```bash
git clone https://github.com/Falplp/mission-control-rs.git
cd mission-control-rs
cargo run --locked
```

For the safest first run, press `m`, select **Offline Demo**, and then press `g`.
The demo provider does not need an API key or a local model server.

Run the project checks with:

```bash
cargo fmt --check
cargo clippy --all-targets -- -D warnings
cargo test --locked --all-targets
```

## Controls

| Key | Action |
| --- | --- |
| `1`–`4`, `j` / `k`, arrows, mouse | Select an agent |
| `Tab`, `←` / `→` | Switch between dashboard views |
| `t` | Submit a task or add it to the queue |
| `l` | Open the pending task queue |
| `g` | Run the selected agent |
| `m` | Select a model and provider |
| `f` | List files in the allowed workspace |
| `v` | Run sandboxed `git status` when terminal access is enabled |
| `o` | Connect to OpenClaw or create an OpenClaw session task |
| `b` | Process tasks from `tasks.txt` |
| `x` | Stop the active OpenClaw session |
| `w` | Change workspace |
| `c` | Change theme |
| `a` | Approve the result in Human Review |
| `r` | Return the task to the generator with feedback |
| `Space` | Pause or resume the demo pipeline |
| `?` / `F1` | Open help |
| `q` / `Esc` | Exit or close the active dialog |

## Configuration

`config.toml` defines agents, roles, models, providers, allowed tools, SOUL files,
workspaces, persistence, OpenClaw, notifications, and security settings. Agents
are created from this configuration instead of being hard-coded in the UI.

Supported providers:

- `demo` — offline and suitable for testing
- `ollama` — local models through `OLLAMA_BASE_URL`
- `openai` — uses `OPENAI_API_KEY`
- `anthropic` — uses `ANTHROPIC_API_KEY`

Copy the environment template before enabling cloud providers or notifications:

```bash
cp .env.example .env
```

Keep secrets in `.env` or the process environment. Never add API keys or gateway
tokens to `config.toml` or commit them to Git.

### Ollama

Start Ollama on the host and download the default generator model:

```bash
ollama serve
ollama pull qwen2.5-coder:14b
```

Local runs use `http://localhost:11434`. Docker Compose uses
`http://host.docker.internal:11434` to reach Ollama on the host.

## Pipeline workflow

Press `t`, enter a task, and confirm with `Enter`. Mission Control then:

1. asks the Architect to create a specification;
2. sends that specification to the Code Generator;
3. writes the generator's `<file path="...">...</file>` artifacts to the allowed workspace;
4. asks the Reviewer to inspect the artifacts;
5. pauses at Human Review for approval or feedback.

Tasks submitted while a run is active are kept in a FIFO queue. Before approval,
the **Artifacts** view shows a diff of files changed by the pipeline.

## Workspace safety

Mission Control can access only the directory configured as
`security.workspace_root`. Absolute paths, `..` traversal, and symbolic links
that escape that root are rejected.

```toml
[security]
workspace_root = "/home/user/projects/my-project"
read_only = false
allow_terminal = false
terminal_allowlist = ["git", "cargo", "openclaw"]
```

For monitoring only, set `read_only = true`. Terminal access is disabled by
default. When enabled, commands run directly without a shell and must be present
in the allowlist.

Optional post-write validation can be configured as an argument array:

```toml
[security]
allow_terminal = true
terminal_allowlist = ["git", "cargo"]
validation_command = ["cargo", "test", "--offline"]
```

## Events and persistence

The application tails `events.jsonl` and reads only newly appended lines. A bad
line appears as an error but does not stop subsequent events.

```json
{"timestamp":"14:35:01","agent":"code-generator","kind":"tool","message":"filesystem.write src/main.py","tokens":12,"duration_ms":240}
```

Pipeline runs, events, errors, and metrics are stored in
`mission_control.sqlite3`. Override the path with `database_path` in the config or
the `MISSION_CONTROL_DB` environment variable.

## OpenClaw Gateway

Mission Control implements the OpenClaw Gateway WebSocket v4 handshake, session
creation, message subscription, streaming, cancellation, and batched task flow.
Keep the gateway token in the process environment:

```bash
openclaw gateway status --deep
read -rsp "OpenClaw token: " OPENCLAW_GATEWAY_TOKEN; echo
export OPENCLAW_GATEWAY_TOKEN
cargo run --locked
```

Press `o` to connect. After **OPENCLAW CONNECTED** appears, press `o` again to
create a session task. Use `x` to stop the active session.

### Batch tasks

Put one task per line in `tasks.txt`; blank lines and lines beginning with `#` are
ignored. Press `b` to process them sequentially. Results are written to
`logs/openclaw-task-results.log` by default.

## Docker

```bash
cp .env.example .env
mkdir -p workspace
docker compose build
docker compose run --rm mission-control
```

The container sees only `./workspace`, mounted at `/workspace`. SQLite data is
kept in a named volume.

## Accessibility

All primary operations are available from the keyboard. Statuses include text
labels instead of relying on color alone, and the UI includes dark, light, and
Starfall themes. Press `?` or `F1` for the in-app key reference.

Terminal screen-reader support varies by terminal emulator. If you encounter an
accessibility barrier, please open an issue and include your operating system,
terminal, and assistive technology.

## Contributing

Bug reports, feature proposals, documentation fixes, and code contributions are
welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

## License

Licensed under the [MIT License](LICENSE).
