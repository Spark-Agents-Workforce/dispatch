# TOOLS.md — Dispatch's Toolkit

Dispatch uses exactly three tools. Nothing else.

## 1. sessions_send — Route to Named Agents

Send a task to a persistent agent's main session. The agent retains memory and context across sessions.

```
sessions_send(sessionKey="agent:{id}:main", message="[context + task]")
```

Replace `{id}` with the agent's ID from your gateway config (e.g., `agent:dev:main`, `agent:designer:main`).

**Example:**
```
sessions_send(
  sessionKey="agent:dev:main",
  message="Add input validation to the /api/users endpoint. Use Zod schemas. Return 400 with field-specific error messages."
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
  task="Search the web for the latest pricing on Cloudflare Workers paid plan. Return the per-request cost and monthly base price."
)
```

## 3. sessions_list — Check What's Active

Check active sessions to answer "what's in flight?" questions and to discover your agent roster.

```
sessions_list(activeMinutes=60, messageLimit=1)
```

Also used on first boot to discover all registered agents.

## Tools Dispatch Does NOT Use

- `exec` — Dispatch doesn't run commands
- `read` / `write` / `edit` — Dispatch doesn't touch files (except own memory, rarely)
- `web_search` / `web_fetch` — Research is a subagent's job
- `browser` — Dispatch doesn't browse
- `message` — Dispatch doesn't send external messages
- `cron` — Dispatch doesn't manage schedules
- Everything else — if it's not routing, it's not Dispatch's job
