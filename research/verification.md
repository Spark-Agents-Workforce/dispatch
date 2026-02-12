# Verification Plan — Dispatch 🚦

**Date:** 2026-02-12
**Author:** Architect 🏛️
**Purpose:** How to verify Dispatch works correctly before going live

---

## Survivor's Gate Results

| # | Gate | Pass | Evidence |
|---|------|------|----------|
| 1 | Standing Purpose | ✅ | Josh will always need an orchestrator. 18 agents require coordination. |
| 2 | Survival Metric | ✅ | 30-second response time. Check: send message, time the routing response. |
| 3 | Domain Depth | ✅ | Extremely narrow: routing only. No work, no research, no creation. |
| 4 | Decision Framework | ✅ | "Named agent → sessions_send. No agent → sessions_spawn. Ambiguous → ask." |
| 5 | Feedback Loop | ✅ | Agent results stream back. Josh corrects routing. Dispatch logs lessons in memory. |
| 6 | Specific Persona | ✅ | Fast, direct, wry. Point guard. Acknowledges then routes. Recognizable in blind test. |
| 7 | Clear Boundaries | ✅ | Complete "NEVER" list in SOUL.md. Comprehensive anti-patterns in AGENTS.md. |
| 8 | Positive Identity | ✅ | Defined by routing speed and context quality. Not "doesn't do work" — "routes everything." |
| 9 | Agent, Not Tool | ✅ | Makes routing decisions, assembles context, tracks in-flight, coordinates parallel work. |
| 10 | Examples Included | ✅ | 5 examples in SOUL.md: simple route, multi-agent, subagent, clarification, status check. |

**All 10 gates pass.**

## Test Scenarios

### Test 1: Simple Named Agent Route
**Send:** "Mason — add CORS headers to the API"
**Expected:** Dispatch acknowledges, sends to `agent:mason:main` with context about Agent Royale API, CORS middleware in Hono. Response in < 30 seconds.
**Verify:** Check sessions_send was called with correct sessionKey.

### Test 2: Subagent (No Named Agent)
**Send:** "What's the current price of ETH?"
**Expected:** Dispatch spawns a disposable subagent. Does NOT try to answer itself or use web_search.
**Verify:** Check sessions_spawn was called, not web_search.

### Test 3: Multi-Agent Parallel
**Send:** "Need brand images from Imager, landing copy from Sarah, tweets from Twin"
**Expected:** Three simultaneous sessions_send calls to agent:studio:main, agent:sarah:main, agent:joshuaday:main. Results reported as they arrive.
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
**Send:** "Have Pixel redesign the hero section"
**Expected:** `sessions_send(sessionKey="agent:pixel:main")` — NOT `sessions_spawn`.
**Verify:** Check dispatch method is sessions_send, not sessions_spawn.

### Test 8: Boot Speed
**Send:** First message in a new session
**Expected:** Response in < 10 seconds. Only SOUL.md + AGENTS.md loaded at boot.
**Verify:** Check tool calls — only 2 read operations at session start.

## Deployment Verification

After registering Dispatch in the gateway config:

1. **Config valid:** `cat ~/.openclaw/openclaw.json | python3 -m json.tool` — no JSON errors
2. **Gateway restart:** `openclaw gateway restart` — clean restart, no errors
3. **Agent responds:** Spawn Dispatch with "Read your SOUL.md. Tell me who you are and list your routing rules."
4. **In character:** Response should be fast, direct, reference the zero-work rule and the 18-agent roster
5. **No work:** Send a research task. Verify it spawns a subagent, doesn't do the work.

## Performance Baseline

Target metrics for Dispatch in production:

| Metric | Target | How to Measure |
|--------|--------|----------------|
| First response latency | < 5 seconds | Time from Josh's message to acknowledgment |
| Routing decision latency | < 30 seconds | Time from message to sessions_send/spawn call |
| Context at boot | < 15KB | SOUL.md (~10KB) + AGENTS.md (~6KB) |
| Files loaded at boot | 2 | SOUL.md + AGENTS.md only |
| Zero-work compliance | 100% | No read/write/exec/web_search/web_fetch tool calls (except memory writes) |

## Rollback Plan

If Dispatch fails in production:

1. Atlas is still registered as `main` in the gateway config
2. Remove Dispatch's SOUL.md from the main agent workspace
3. Restore Atlas's SOUL.md
4. Restart gateway
5. Document what went wrong for Architect to fix
