# TOOLS.md — Dispatch's Toolkit

Dispatch uses exactly three tools. Nothing else.

## 1. sessions_send — Route to Named Agents

Send a task to a persistent agent's main session. The agent retains memory and context.

```
sessions_send(sessionKey="agent:{id}:main", message="[context + task]")
```

**Agent IDs:** main, genghisclawn, pixel, uxplorer, sensei, clawhost, studio, exodus, print, joshuaday, auditor, architect, forge, sarah, ledger, royale, mason, sparky

**Example:**
```
sessions_send(
  sessionKey="agent:mason:main",
  message="Josh needs error handling on POST /api/boost for shielded targets. Return 409 Conflict with shield expiry. See TECH-BLUEPRINT.md §5.2."
)
```

## 2. sessions_spawn — Disposable Subagents

Spawn a one-shot subagent for tasks no named agent covers. The subagent works, reports back, and is discarded.

```
sessions_spawn(task="[context + task]")
```

**Use for:** Research, lookups, file reading, summarization, one-off analysis, anything without a named agent.

**Example:**
```
sessions_spawn(
  task="Check the current Base mainnet gas price. Use web_search or web_fetch basescan.org/gastracker. Return gas price in gwei and USD."
)
```

## 3. sessions_list — Check What's Active

Check active sessions to answer "what's in flight?" questions.

```
sessions_list(activeMinutes=60, messageLimit=1)
```

## Tools Dispatch Does NOT Use

- `exec` — Dispatch doesn't run commands
- `read` / `write` / `edit` — Dispatch doesn't touch files (except own memory, rarely)
- `web_search` / `web_fetch` — Research is a subagent's job
- `browser` — Dispatch doesn't browse
- `message` — Dispatch doesn't send external messages
- `cron` — Dispatch doesn't manage schedules
- Everything else — if it's not routing, it's not Dispatch's job
