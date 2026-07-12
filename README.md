# pi-grok-sdk

Use **Grok Build** inside [pi](https://github.com/badlogic/pi-mono) without giving up the agent that already knows your machine.

`pi-grok-sdk` wires pi to the local xAI **agent CLI** (`agent` / `grok`). You get pi’s model picker, sessions, and streaming UI; Grok keeps its tools, skills, MCP servers, permissions, and multi-turn loop. Inference is not a thin HTTP chat wrapper — it’s the real local agent runtime, streamed back as native pi events.

**Models:** `grok-sdk/grok-4.5` (and others from `agent models`)

## Requirements

- pi 0.80.6+
- Node 22.19+
- Grok agent CLI installed and logged in

```bash
which agent || which grok
agent --version
agent models
```

If the binary only lives under `~/.grok/bin`, add it to `PATH` or set `PI_GROK_SDK_BIN`.

## Install

```bash
pi install npm:pi-grok-sdk
```

Alternatives:

```bash
pi install https://github.com/ankitchouhan1020/pi-grok-sdk
pi install /path/to/pi-grok-sdk   # local checkout
```

```bash
pi remove npm:pi-grok-sdk
```

## Quick start

```bash
pi --list-models grok-sdk

pi --model grok-sdk/grok-4.5
pi --model grok-sdk/grok-4.5 -p "Say hi"
pi --model grok-sdk/grok-4.5 --thinking high -p "…"
```

Smoke test:

```bash
pi --model grok-sdk/grok-4.5 --no-session -p \
  "Reply exactly PI_GROK_OK and nothing else."
```

Optional defaults in `~/.pi/agent/settings.json`:

```json
{
  "defaultProvider": "grok-sdk",
  "defaultModel": "grok-4.5"
}
```

### Commands

| Command | What it does |
| --- | --- |
| `/grok-sdk status` | Binary, auth, mode, models |
| `/grok-sdk models` | Live CLI model list |
| `/grok-sdk mode` | Current mode + env hints |
| `/grok-sdk refresh-models` | Re-discover models |

`/grok-agent` is an alias for `/grok-sdk`.

## How it works

```text
pi  ──streamSimple──►  pi-grok-sdk
                          │
                          ├─ acp (default)  long-lived agent agent … stdio
                          │                 multi-turn session pool
                          │                 thinking → text (native pi events)
                          │
                          └─ jsonl          one-shot --single streaming-json
```

**ACP (default)** keeps a long-lived Grok process per pi session (model + cwd + effort). Turns speak ACP over stdio; later messages only send new user content so Grok retains history and tools.

**JSONL** is a one-shot fallback for debugging:

```bash
PI_GROK_SDK_MODE=jsonl pi --model grok-sdk/grok-4.5 -p "…"
```

Streaming is pi-native: token deltas, thinking closed before the answer, and event-loop yields so the TUI can paint mid-stream.

## Config

| Env | Default | Meaning |
| --- | --- | --- |
| `PI_GROK_SDK_MODE` | `acp` | `acp` or `jsonl` |
| `PI_GROK_SDK_BIN` | — | Path to `agent` / `grok` |
| `PI_GROK_SDK_SHOW_TOOLS` | off | Show Grok tool titles as short thinking notes |

Aliases still work: `PI_GROK_AGENT_MODE`, `PI_GROK_AGENT_BIN`, etc.

Binary lookup: env → `PATH` (`agent`, `grok`) → `~/.grok/bin`.

Pi `--thinking` maps to Grok `--reasoning-effort` (`off` / `minimal` / `low` / `medium` / `high` / `xhigh` / `max`).

## Compat

- Legacy provider id: `grok-agent-cli/…` (same models)
- Legacy model id: `grok-4.5-agent` → `grok-4.5`
- Text-only input for now

## Not in scope

- Pi tools are not bridged into Grok (Grok runs its own tools)
- Not an xAI HTTP / OpenAI-compatible API
- No image input yet

## Troubleshooting

| Problem | Fix |
| --- | --- |
| No models | Check package path in settings; restart pi |
| CLI not found | `which agent` or `PI_GROK_SDK_BIN` |
| Auth errors | Run `agent` / `grok login` once in a terminal |
| Stale models | `/grok-sdk refresh-models` |
