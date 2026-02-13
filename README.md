# Dispatch 🚦 — Universal AI Orchestrator for OpenClaw

Dispatch is a drop-in orchestrator agent for [OpenClaw](https://github.com/openclaw/openclaw). It's a pure non-blocking router based on the 911 dispatcher model: receive, acknowledge, route, report. Zero work. Ever.

**Works with any fleet.** Dispatch auto-discovers your agents from the gateway config — no hardcoded names, no manual routing tables. Whether you have 3 agents or 30, Dispatch learns your roster and starts routing.

## Why Dispatch Exists

Most orchestrator agents fail because they start doing work instead of routing. Research tasks, file summaries, planning sessions — the orchestrator gets busy and the user is locked out. See `research/atlas-autopsy.md` for a real post-mortem of this failure mode.

Dispatch fixes this with one absolute rule: **the dispatcher never leaves the switchboard.**

## Architecture

```
         User (iMessage / Slack / webchat / any channel)
                    │
                    ▼
            ┌──────────────┐
            │  Dispatch 🚦  │
            │  (30s max)    │
            └──────┬───────┘
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
  sessions_send  sessions_send  sessions_spawn
  (named agent)  (named agent)  (disposable)
        │          │          │
   agent:X:main  agent:Y:main  one-shot task
   (has memory)  (has memory)  (no memory)
```

## The Two Dispatch Methods

This is the critical design decision:

### sessions_send → Named Agents (Persistent)
```
sessions_send(sessionKey="agent:dev:main", message="...")
```
Agent wakes in its main session. Has access to MEMORY.md, daily notes, full project context. Use for ALL named agents in your fleet.

### sessions_spawn → Subagents (Disposable)
```
sessions_spawn(task="...")
```
Fresh session, no memory, no prior context. Use for one-shot research, lookups, throwaway tasks.

**Rule:** Named agents ALWAYS get sessions_send. NEVER spawn a named agent as a disposable subagent.

## Quick Start

### 1. Copy agent files to your OpenClaw

```bash
# Clone this repo
git clone https://github.com/Spark-Agents-Workforce/dispatch.git
cd dispatch

# Copy to OpenClaw agent config (Dispatch replaces your main agent)
mkdir -p ~/.openclaw/agents/main
cp SOUL.md AGENTS.md IDENTITY.md USER.md TOOLS.md ~/.openclaw/agents/main/

# Also copy to workspace
cp SOUL.md AGENTS.md IDENTITY.md USER.md TOOLS.md ~/.openclaw/workspace/
```

### 2. Fill in USER.md

Edit `~/.openclaw/agents/main/USER.md` (and `~/.openclaw/workspace/USER.md`) with your name, timezone, and communication preferences. This is the only file you need to customize.

### 3. Update gateway config identity

Edit `~/.openclaw/openclaw.json` — change the main agent identity:
```json
{
  "id": "main",
  "identity": {
    "name": "Dispatch",
    "emoji": "🚦"
  }
}
```

### 4. Restart

```bash
openclaw gateway restart
```

### 5. Verify

Send Dispatch a message: "Who are you? What agents do I have?"

Dispatch should:
- Identify itself (fast, direct, references the zero-work rule)
- Run `sessions_list` to discover your agent roster
- Store the roster in MEMORY.md for future sessions

## Files

| File | Purpose | Customize? |
|------|---------|------------|
| `SOUL.md` | Identity, zero-work rule, routing logic, examples | No |
| `AGENTS.md` | Boot sequence, routing loop, anti-patterns, first-boot discovery | No |
| `IDENTITY.md` | Name, emoji, vibe | No |
| `USER.md` | Your info — name, timezone, preferences | **Yes** |
| `TOOLS.md` | 3 tools: sessions_send, sessions_spawn, sessions_list | No |
| `README.md` | This file | — |
| `research/` | Design rationale, pattern research, verification plan | — |

## How Auto-Discovery Works

Dispatch does not ship with a routing table. On first boot:

1. Calls `sessions_list` to find all registered agents
2. Reads each agent's `IDENTITY.md` to learn their domain
3. Builds a routing table (name → ID → domain → dispatch method)
4. Stores the table in `MEMORY.md`

On subsequent boots, Dispatch loads its routing table from MEMORY.md. If the user mentions an agent Dispatch doesn't recognize, it re-runs discovery.

This means Dispatch works with ANY OpenClaw fleet — no configuration beyond USER.md.

## Design Decisions

| Decision | Choice | Reasoning |
|----------|--------|-----------|
| Pattern | 911 Dispatcher | Zero-work guarantee. See `research/orchestration-patterns.md` |
| Roster | Auto-discovered | Works for any fleet. No hardcoded agent names. |
| Boot files | 2 (SOUL + AGENTS) | Fast first response. Everything else on demand |
| Named agents | sessions_send always | Preserve memory/context. See `research/atlas-autopsy.md` |
| Response ceiling | 30 seconds | If it takes longer, you're doing work |
| Model | Inherits from global config | Routing needs intelligence for context assembly |
| Heartbeat | None | Purely reactive — responds when the user messages |
| Tools | 3 only | sessions_send, sessions_spawn, sessions_list |

## Research

The `research/` folder contains the design rationale:
- **atlas-autopsy.md** — Post-mortem of a failed orchestrator (why zero-work is non-negotiable)
- **orchestration-patterns.md** — Survey of 5 patterns, why 911 dispatcher wins
- **name-options.md** — Why "Dispatch" (the name encodes the constraint)
- **verification.md** — Survivor's Gate results, test scenarios, performance baselines

## License

MIT
