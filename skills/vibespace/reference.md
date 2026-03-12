# vibespace CLI Reference

## vibespace init

Boot a local Kubernetes (k3s) cluster.

```
--cpu <cores>          CPU cores for VM (default: 4)
--memory <GiB>         Memory for VM (default: 8)
--disk <GiB>           Disk size for VM (default: 60)
--bare-metal           Skip VM, install k3s directly (Linux only, needs root)
--external             Use existing Kubernetes cluster
--kubeconfig <path>    Path to external cluster kubeconfig
```

macOS: starts Colima VM. Linux: starts Lima VM with QEMU (or `--bare-metal` for native k3s).

## vibespace create

Create a vibespace with its first agent.

```
vibespace create <name> --agent-type <claude-code|codex> [flags]

-t, --agent-type       Agent type: claude-code, codex (required)
-s, --share-credentials  Shared credential store across agents of same type
--skip-permissions     Agent runs with full tool access (no prompts)
--repo <url>           Clone GitHub repo into workspace
--worktree             Enable git worktree mode (each agent gets own branch)
--branch <name>        Git branch for initial agent
--mount <src:dst>      Mount local directory (append :ro for read-only)
--cpu <millicores>     CPU request (e.g., 500m)
--cpu-limit <m>        CPU burst limit
--memory <size>        Memory request (e.g., 1Gi)
--memory-limit <size>  Memory burst limit
--storage <size>       Persistent volume size (e.g., 20Gi)
```

## vibespace agent create

Add an agent to an existing vibespace.

```
vibespace agent create --vibespace <name> [flags]

-t, --agent-type       Agent type: claude-code, codex
-n, --name <name>      Agent name (auto-generated if omitted: claude-1, codex-1)
-s, --share-credentials
--branch <name>        Git branch (worktree mode only)
--model <model>        Model: opus, sonnet, haiku (Claude) or varies (Codex)
--max-turns <n>        Max conversation turns
--allowed-tools <list>   Comma-separated allowed tools
--disallowed-tools <list>
--skip-permissions
--system-prompt <text>   Custom system prompt
--reasoning-effort <level>  Codex: low, medium, high, xhigh
```

## vibespace agent start/stop/delete

```
vibespace agent start [agent-name] --vibespace <name>   # start all or specific agent
vibespace agent stop [agent-name] --vibespace <name>    # stop all or specific agent
vibespace agent delete <agent-name> --vibespace <name>
vibespace agent list --vibespace <name> [--json]
```

## vibespace connect

SSH into an agent container.

```
vibespace connect --vibespace <name> [-a <agent>] [-b]

-a, --agent <name>     Target agent (default: primary)
-b, --browser          Open web terminal (ttyd) instead of SSH
```

## vibespace exec

Run a command inside an agent container.

```
vibespace exec --vibespace <name> [-a <agent>] -- <command>
```

## vibespace multi

Multi-agent chat sessions.

```
# Interactive TUI session
vibespace multi --vibespaces <vs1,vs2> [--name <session>] [--agents a@vs1,b@vs2]

# Non-interactive: send message, get response
vibespace multi "message" --vibespaces <vs> --agent <agent> [--stream] [--timeout 2m]

# Batch mode: read JSONL from stdin
vibespace multi --batch --vibespaces <vs>

# Resume session
vibespace multi --resume [session-id]

# List sessions
vibespace multi --list-sessions [--json]
```

Interactive TUI slash commands:
- `@agent-name message` — target specific agent
- `@all message` — broadcast to all
- `/list` — show connected agents
- `/add <vibespace>` or `/add agent@vibespace` — join agent to session
- `/remove <vibespace>` — leave session
- `/focus <agent>` — solo view
- `/clear` — clear chat
- `/session` — session info
- `/scroll` — toggle auto-scroll
- `/help` — show commands
- `/quit` — exit

## vibespace config

View and modify agent settings.

```
vibespace config show [agent] --vibespace <name> [--json]
vibespace config set <agent> --vibespace <name> [flags]

Flags: --model, --max-turns, --allowed-tools, --disallowed-tools,
       --skip-permissions, --system-prompt, --reasoning-effort
```

## vibespace forward

Port forwarding with optional DNS.

```
vibespace forward add <port> --vibespace <name> [-a <agent>] [-l <local-port>] [--dns] [--dns-name <name>]
vibespace forward list --vibespace <name> [--json]
vibespace forward remove <port> --vibespace <name> [-a <agent>]
```

DNS creates `<name>.vibespace.internal` entries resolved via embedded DNS server.

## vibespace remote

WireGuard VPN for remote cluster access.

```
# Server (on VPS)
vibespace serve [--foreground] [--endpoint <IP>]
vibespace serve --generate-token [--token-ttl 1h]
vibespace serve --list-clients
vibespace serve --remove-client <name>

# Client
sudo vibespace remote connect <token>
vibespace remote status
vibespace remote watch          # auto-reconnect on drops
vibespace remote disconnect
```

## vibespace session

Manage saved multi-agent sessions.

```
vibespace session list [--json]
vibespace session show <session-id>
vibespace session delete <session-id>
```

## Other commands

```
vibespace list [--json] [--plain]
vibespace info --vibespace <name> [--json]
vibespace status [--json]
vibespace stop
vibespace delete <name> [--force] [--keep-data] [--dry-run]
vibespace tui
vibespace upgrade [--check] [--force]
vibespace uninstall [--force]
vibespace version
```

## Environment variables

| Variable | Purpose |
|----------|---------|
| `VIBESPACE_NAME` | Default vibespace (instead of `--vibespace`) |
| `VIBESPACE_DEBUG=1` | Debug logging to ~/.vibespace/debug.log |
| `VIBESPACE_LOG_LEVEL` | debug, info, warn, error |
| `VIBESPACE_CLUSTER_CPU` | Default cluster CPU cores |
| `VIBESPACE_CLUSTER_MEMORY` | Default cluster memory |
| `VIBESPACE_CLUSTER_DISK` | Default cluster disk |
| `VIBESPACE_NO_UPDATE_CHECK` | Disable startup update check |
| `VIBESPACE_CONFIG` | Custom config file path |
| `NO_COLOR` | Disable colored output |

## Output modes

- **TTY** → human-readable tables with colors
- **Piped/non-TTY** → auto-switches to `--json`
- `--json` → `{success, data, error, meta}` envelope
- `--plain --header` → TSV with column headers

## Exit codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Internal error |
| 2 | Invalid usage |
| 10 | Not found |
| 11 | Already exists |
| 12 | Permission denied |
| 13 | Timeout |
| 14 | Cancelled |
| 15 | Service unavailable |
