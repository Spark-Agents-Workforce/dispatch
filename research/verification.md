# Verification Plan — Dispatch 🚦

**Purpose:** How to verify Dispatch works correctly before going live

---

## Survivor's Gate Results

| # | Gate | Pass | Evidence |
|---|------|------|----------|
| 1 | Standing Purpose | ✅ | Any multi-agent fleet needs coordination. Dispatch exists as long as agents do. |
| 2 | Survival Metric | ✅ | 30-second response time. Check: send message, time the routing response. |
| 3 | Domain Depth | ✅ | Extremely narrow: routing only. No work, no research, no creation. |
| 4 | Decision Framework | ✅ | "Named agent → sessions_send. No agent → sessions_spawn. Ambiguous → ask." |
| 5 | Feedback Loop | ✅ | Agent results stream back. User corrects routing. Dispatch logs lessons in memory. |
| 6 | Specific Persona | ✅ | Fast, direct, wry. Point guard. Acknowledges then routes. Recognizable in blind test. |
| 7 | Clear Boundaries | ✅ | Complete "NEVER" list in SOUL.md. Comprehensive anti-patterns in AGENTS.md. |
| 8 | Positive Identity | ✅ | Defined by routing speed and context quality. Not "doesn't do work" — "routes everything." |
| 9 | Agent, Not Tool | ✅ | Makes routing decisions, assembles context, tracks in-flight, coordinates parallel work. |
| 10 | Examples Included | ✅ | 5 examples in SOUL.md: simple route, multi-agent, subagent, clarification, status check. |

**All 10 gates pass.**

## Test Scenarios

### Test 1: Simple Named Agent Route
**Send:** "[Your coding agent] — add input validation to the API"
**Expected:** Dispatch acknowledges, sends to `agent:{id}:main` with context. Response in < 30 seconds.
**Verify:** Check sessions_send was called with correct sessionKey.

### Test 2: Subagent (No Named Agent)
**Send:** "What's the current price of Bitcoin?"
**Expected:** Dispatch spawns a disposable subagent. Does NOT try to answer itself or use web_search.
**Verify:** Check sessions_spawn was called, not web_search.

### Test 3: Multi-Agent Parallel
**Send:** "Need brand images, landing copy, and announcement tweets"
**Expected:** Three simultaneous sessions_send calls. Results reported as they arrive.
**Verify:** All three sent within the same response. Not serialized.

### Test 4: The Temptation Test
**Send:** "Summarize the last 5 entries in MEMORY.md"
**Expected:** Dispatch spawns a subagent. Does NOT read MEMORY.md itself.
**Verify:** No `read` tool call. Only sessions_spawn.

### Test 5: Ambiguity
**Send:** "Fix the button"
**Expected:** Dispatch asks ONE clarifying question about which project/agent.
**Verify:** No routing attempt. Just a question.

### Test 6: Status Query (Dispatch's Domain)
**Send:** "What's in flight?"
**Expected:** Dispatch answers directly from its own knowledge. No agent routing needed.
**Verify:** Direct response, no sessions_send or sessions_spawn.

### Test 7: Named Agent, Not Subagent
**Send:** "Have [design agent] redesign the hero section"
**Expected:** `sessions_send(sessionKey="agent:{id}:main")` — NOT `sessions_spawn`.
**Verify:** Check dispatch method is sessions_send, not sessions_spawn.

### Test 8: Boot Speed
**Send:** First message in a new session
**Expected:** Response in < 10 seconds. Only SOUL.md + AGENTS.md loaded at boot.
**Verify:** Check tool calls — only 2 read operations at session start.

### Test 9: Auto-Discovery
**Send:** "What agents do I have?" (on first boot, before MEMORY.md exists)
**Expected:** Dispatch runs sessions_list, discovers roster, stores in MEMORY.md, reports back.
**Verify:** sessions_list called, MEMORY.md created with routing table.

## Deployment Verification

After installing Dispatch:

1. **Config valid:** `cat ~/.openclaw/openclaw.json | python3 -m json.tool` — no JSON errors
2. **Gateway restart:** `openclaw gateway restart` — clean restart, no errors
3. **Agent responds:** Send "Who are you?"
4. **In character:** Response should be fast, direct, reference the zero-work rule
5. **Discovery works:** Send "List my agents" — Dispatch discovers roster via sessions_list
6. **No work:** Send a research task — verify it spawns a subagent, doesn't do the work

## Performance Baseline

| Metric | Target | How to Measure |
|--------|--------|----------------|
| First response latency | < 5 seconds | Time from message to acknowledgment |
| Routing decision latency | < 30 seconds | Time from message to sessions_send/spawn call |
| Context at boot | < 15KB | SOUL.md (~8KB) + AGENTS.md (~5KB) |
| Files loaded at boot | 2 | SOUL.md + AGENTS.md only |
| Zero-work compliance | 100% | No read/write/exec/web_search/web_fetch tool calls (except memory writes) |

## Rollback Plan

If Dispatch doesn't work for your setup:

1. Remove Dispatch's files from `~/.openclaw/agents/main/`
2. Restore your previous orchestrator's SOUL.md
3. Restart gateway: `openclaw gateway restart`
4. Note what went wrong — consider opening an issue on the repo
