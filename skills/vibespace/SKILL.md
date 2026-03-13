---
name: vibespace
description: Create and manage stateful runtime environments for AI agents with vibespace. Use when the user wants persistent agent environments, sandboxed AI coding agents, multi-agent sessions, remote shared clusters, or mentions "vibespace".
---

# vibespace

Stateful runtime environments for AI agents. Each agent gets a persistent Linux container — its own filesystem, tools, network, and identity — orchestrated by Kubernetes on a local cluster or remote server.

## Quick start

```bash
vibespace init                                                            # boot local k3s cluster
vibespace create my-project --agent-type claude-code --share-credentials  # create vibespace + agent
vibespace connect --vibespace my-project                                  # SSH into the agent's container
```

## Creating vibespaces and agents

```bash
# Claude Code agent with a GitHub repo
vibespace create my-project -t claude-code -s --repo https://github.com/org/repo

# Codex agent
vibespace create my-project -t codex -s --repo https://github.com/org/repo

# Mount a local directory
vibespace create my-project -t claude-code -s --mount ~/code/my-repo:/vibespace

# Worktree mode: each agent gets its own git branch
vibespace create my-project -t claude-code -s --repo https://github.com/org/repo --worktree

# Add another agent to an existing vibespace
vibespace agent create --vibespace my-project -t claude-code --branch feat/auth -s
```

Key flags:
- `-t claude-code` or `-t codex` — agent type
- `-s` / `--share-credentials` — shared credential store across agents of same type
- `--skip-permissions` — full autonomy, no permission prompts
- `--worktree` — git worktree mode (each agent gets own branch)

## Agent configuration

Tunable per agent at any time:

```bash
vibespace config set claude-1 --vibespace my-project \
  --model opus \
  --max-turns 50 \
  --skip-permissions \
  --system-prompt "You are a backend engineer focused on API design"
```

Options: `--model`, `--max-turns`, `--allowed-tools`, `--disallowed-tools`, `--skip-permissions`, `--system-prompt`, `--reasoning-effort` (Codex only).

## Multi-agent sessions

```bash
# Interactive TUI session
vibespace multi --vibespaces my-project

# Send a message non-interactively
vibespace multi "implement the auth module" --vibespaces my-project --plain

# Stream response token-by-token
vibespace multi "explain the codebase" --vibespaces my-project --stream

# Resume a named session
vibespace multi --resume my-session "next task"
```

In TUI: `@agent-name message` to target, `@all message` to broadcast, `/focus <agent>` for solo view.

## Interacting with agents

```bash
vibespace connect --vibespace my-project                    # SSH into agent container
vibespace connect --vibespace my-project -a claude-2        # SSH into specific agent
vibespace exec --vibespace my-project -- cat /vibespace/CLAUDE.md  # run command in container
```

## Port forwarding

```bash
vibespace forward add 8080 --vibespace my-project --dns     # forward + DNS entry
# Creates <agent>.vibespace.internal:8080
```

## Remote mode

Run the cluster on a VPS and share with teammates via WireGuard VPN:

```bash
# Server
vibespace init --bare-metal
vibespace serve
vibespace serve --generate-token --token-ttl 24h

# Client (from any machine)
sudo vibespace remote connect <token>
```

## Key commands

| Command | Purpose |
|---------|---------|
| `vibespace init` | Boot k3s cluster |
| `vibespace create <name> -t <type>` | Create vibespace with agent |
| `vibespace agent create --vibespace <vs>` | Add agent to vibespace |
| `vibespace connect --vibespace <vs>` | SSH into agent container |
| `vibespace exec --vibespace <vs> -- <cmd>` | Run command in container |
| `vibespace multi --vibespaces <vs>` | Multi-agent chat session |
| `vibespace config set <agent> --vibespace <vs>` | Change agent settings |
| `vibespace forward add <port> --vibespace <vs>` | Port forward |
| `vibespace list` | List all vibespaces |
| `vibespace info --vibespace <vs>` | Vibespace details |
| `vibespace status` | Cluster health |
| `vibespace tui` | Terminal UI |
| `vibespace remote connect <token>` | Connect to remote cluster |
| `vibespace stop` | Stop cluster (preserves data) |
| `vibespace upgrade` | Self-update |

## Installation

```bash
brew install vibespacehq/tap/vibespace                    # macOS (Homebrew)
curl -fsSL https://raw.githubusercontent.com/vibespacehq/vibespace/main/install.sh | bash  # any platform
go install github.com/vibespacehq/vibespace/cmd/vibespace@latest  # Go 1.25+
npx skills add vibespacehq/vibespace-skill                # agent skill
```

## Documentation

Detailed docs in `references/`:

- [reference.md](reference.md) — complete CLI flags, env vars, output modes, exit codes
- [references/getting-started.md](references/getting-started.md) — install, init, create, connect
- [references/concepts.md](references/concepts.md) — vibespaces, agents, sessions, remote mode
- [references/cli-reference.md](references/cli-reference.md) — every command and flag
- [references/configuration.md](references/configuration.md) — config file, resources, theming
- [references/multi-agent.md](references/multi-agent.md) — multi-agent sessions, targeting, batch
- [references/remote-mode.md](references/remote-mode.md) — WireGuard VPN setup, tokens
- [references/port-forwarding.md](references/port-forwarding.md) — port forwards, DNS
- [references/tui.md](references/tui.md) — terminal UI tabs, keybindings
- [references/installation.md](references/installation.md) — system requirements, all methods
- [references/troubleshooting.md](references/troubleshooting.md) — common problems, debug logging

Full docs: https://vibespace.build
