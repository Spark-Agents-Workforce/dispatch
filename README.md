# Dispatch 🚦 — The Orchestrator v2

Dispatch replaces Atlas as the main orchestrator for Josh's 18-agent AI workforce. It's a pure non-blocking router based on the 911 dispatcher model: receive, acknowledge, route, report. Zero work. Ever.

## Why Dispatch Exists

Atlas (the previous orchestrator) failed because it kept doing work instead of routing. Research tasks, file summaries, planning sessions — all locked Josh out while Atlas was busy. See `research/atlas-autopsy.md` for the full post-mortem.

Dispatch fixes this with one absolute rule: **the dispatcher never leaves the switchboard.**

## Architecture

```
         Josh (iMessage / Slack / webchat)
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
sessions_send(sessionKey="agent:mason:main", message="...")
```
Agent wakes in its main session. Has access to MEMORY.md, daily notes, full project context. Use for ALL 18 named agents.

### sessions_spawn → Subagents (Disposable)
```
sessions_spawn(task="...")
```
Fresh session, no memory, no prior context. Use for one-shot research, lookups, throwaway tasks.

**Rule:** Named agents ALWAYS get sessions_send. NEVER spawn a named agent as a disposable subagent.

## The 18-Agent Roster

| Agent | ID | Domain |
|-------|-----|--------|
| Atlas 🔱 | `main` | Legacy (only if explicitly requested) |
| Khan 🦞 | `genghisclawn` | X/Twitter for OpenClaw |
| Pixel 🎨 | `pixel` | Web design, brand sites |
| Iris 👁️ | `uxplorer` | UX/UI, Orchestrator V3 |
| Sensei 🥋 | `sensei` | Tutorials, docs |
| ClawHost 🏗️ | `clawhost` | Heavy coding, TryClaw |
| Imager 🖼️ | `studio` | Image generation |
| Exodus 🔓 | `exodus` | Memory liberation |
| Printer 💵 | `print` | Trading, markets |
| Twin ⚡ | `joshuaday` | X/Twitter as Josh |
| Auditor 🔍 | `auditor` | QA, testing |
| Architect 🏛️ | `architect` | Agent design |
| Forge 🔥 | `forge` | Product strategy |
| Sarah ✒️ | `sarah` | PR, communications |
| Ledger 📉 | `ledger` | Cost analysis |
| Royale 👑 | `royale` | Agent Royale PM |
| Mason 🧱 | `mason` | Agent Royale dev |
| Sparky ⚡ | `sparky` | Customer support |

## Files

| File | Purpose |
|------|---------|
| `SOUL.md` | Identity, zero-work rule, routing table, 5 examples |
| `AGENTS.md` | Boot sequence, routing loop, anti-patterns, escalation |
| `IDENTITY.md` | Name, emoji, vibe |
| `USER.md` | Josh's needs from the orchestrator |
| `TOOLS.md` | 3 tools only: sessions_send, sessions_spawn, sessions_list |
| `README.md` | This file |
| `research/atlas-autopsy.md` | What went wrong with Atlas |
| `research/orchestration-patterns.md` | Why 911 dispatcher pattern wins |
| `research/name-options.md` | Name evaluation (why "Dispatch") |
| `research/verification.md` | Survivor's Gate results + test scenarios |

## Deployment

### 1. Copy files to main agent workspace
```bash
# Dispatch replaces Atlas in the main slot
cp v2/SOUL.md ~/.openclaw/agents/main/SOUL.md
cp v2/AGENTS.md ~/.openclaw/agents/main/AGENTS.md
cp v2/IDENTITY.md ~/.openclaw/agents/main/IDENTITY.md
cp v2/USER.md ~/.openclaw/agents/main/USER.md
cp v2/TOOLS.md ~/.openclaw/agents/main/TOOLS.md

# Also copy to workspace
cp v2/SOUL.md ~/.openclaw/workspace/SOUL.md
cp v2/AGENTS.md ~/.openclaw/workspace/AGENTS.md
cp v2/IDENTITY.md ~/.openclaw/workspace/IDENTITY.md
cp v2/USER.md ~/.openclaw/workspace/USER.md
cp v2/TOOLS.md ~/.openclaw/workspace/TOOLS.md
```

### 2. Update gateway config identity
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

### 3. Restart
```bash
openclaw gateway restart
```

### 4. Verify
Send: "Who are you? List your routing rules."
Expected: Fast, direct response identifying as Dispatch, referencing zero-work rule and 18-agent roster.

## Design Decisions

| Decision | Choice | Reasoning |
|----------|--------|-----------|
| Pattern | 911 Dispatcher | Zero-work guarantee. See `research/orchestration-patterns.md` |
| Boot files | 2 (SOUL + AGENTS) | Fast first response. Everything else on demand |
| Named agents | sessions_send always | Preserve memory/context. See `research/atlas-autopsy.md` |
| Response ceiling | 30 seconds | If it takes longer, you're doing work |
| Model | Inherits global default (Opus 4.6) | Routing needs intelligence for context assembly |
| Heartbeat | None | Purely reactive — responds when Josh messages |
| Tools | 3 only | sessions_send, sessions_spawn, sessions_list |
