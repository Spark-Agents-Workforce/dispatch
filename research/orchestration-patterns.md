# Orchestration Patterns — Research for Dispatch Design

**Date:** 2026-02-12
**Author:** Architect 🏛️
**Purpose:** Survey of orchestration models and why the 911 dispatcher pattern wins

---

## Patterns Considered

### 1. Manager Pattern (Atlas v1)
The orchestrator is a smart manager who can do work when needed and delegates when appropriate.

**Pros:** Flexible, can handle edge cases, feels natural
**Cons:** "When needed" always expands. Manager becomes bottleneck. The very flexibility is the failure mode.

**Verdict:** ❌ Failed in production. Atlas proved that a manager who CAN do work WILL do work.

### 2. Router Pattern (Pure)
Stateless request router. Maps input to agent, forwards, done. No intelligence, no context, no memory.

**Pros:** Fast, simple, never does work
**Cons:** Dumb routing leads to bad context assembly. Agent receives "fix it" with no details. Quality of delegation suffers.

**Verdict:** ❌ Too dumb. Routing without context assembly is just forwarding email.

### 3. 911 Dispatcher Pattern (Dispatch v2) ✅
Intelligent routing with zero execution. The dispatcher understands the request, assembles context, routes to the right responder, and tracks status. But NEVER leaves the switchboard.

Key properties:
- **Zero execution** — the dispatcher never goes on the call
- **Intelligent routing** — understands the request well enough to pick the right responder
- **Context assembly** — provides the responder with everything they need
- **Status tracking** — knows what's in flight, reports back
- **Always available** — because they never leave the desk

**Pros:** Combines routing intelligence with zero-work guarantee. The hard boundary prevents scope creep. Always available to Josh.
**Cons:** Requires discipline in the SOUL.md to maintain the boundary. Slightly more latency than a pure router (context assembly takes a few seconds).

**Verdict:** ✅ The right pattern. Intelligent enough to be useful, constrained enough to never bottleneck.

### 4. Pub/Sub Pattern
Events published to a bus, agents subscribe to topics they handle. No central orchestrator.

**Pros:** Fully decentralized, scales infinitely, no single point of failure
**Cons:** No one assembles context. No one tracks status. No one answers "what's happening?" Josh loses visibility.

**Verdict:** ❌ Loses the human-in-the-loop. Josh needs a single point of contact.

### 5. Hierarchical Pattern
Multi-level management. Dispatch → Department leads → Individual agents.

**Pros:** Scales to hundreds of agents, natural decomposition
**Cons:** Overkill for 18 agents. Adds latency. More points of failure.

**Verdict:** ❌ For now. May be relevant at 50+ agents.

## The Critical Insight: sessions_send vs sessions_spawn

The single most important implementation detail in the entire orchestration design.

### sessions_send(sessionKey="agent:{id}:main")
- Sends to the agent's **persistent main session**
- Agent has access to its MEMORY.md, daily notes, prior conversation context
- Agent remembers what it worked on yesterday
- Appropriate for: named specialist agents with ongoing responsibilities

### sessions_spawn(task="...")
- Creates a **new, disposable session**
- No memory, no prior context, fresh start
- Results reported back, session can be discarded
- Appropriate for: one-shot tasks, research, lookups, anything throwaway

### Why This Matters
When Mason is working on Agent Royale and you spawn a new session for a follow-up task, Mason starts from zero. Doesn't know the codebase decisions, doesn't remember the pricing engine implementation, doesn't know which endpoints are done. All that context — gone.

When you `sessions_send` to `agent:mason:main`, Mason wakes up with full context. Reads MEMORY.md, sees yesterday's work, knows the state of the project. Can pick up exactly where it left off.

**This distinction is the difference between a team of colleagues and a pool of amnesiacs.**

## Parallel Execution Model

The dispatcher's superpower: launching multiple agents simultaneously.

```
Josh: "Launch prep — images, copy, and tweets"

WRONG (serial):
  t=0:  spawn Imager → wait 3 min → result
  t=3:  spawn Sarah → wait 3 min → result
  t=6:  spawn Twin → wait 2 min → result
  Total: 8 minutes

RIGHT (parallel):
  t=0:  send to Imager, Sarah, Twin simultaneously
  t=2:  Twin finishes → report to Josh
  t=3:  Imager finishes → report to Josh
  t=3:  Sarah finishes → report to Josh
  Total: 3 minutes
```

Report results as they arrive. Don't wait for all to finish.

## Dependent Task Chains

Some tasks have dependencies: A must finish before B can start.

```
Josh: "Design the new logo, then use it on the landing page"

  t=0:  send to Imager for logo
  t=3:  Imager finishes → report to Josh → send to Pixel with logo output
  t=6:  Pixel finishes → report to Josh
```

Dispatch tracks the dependency and triggers the next step automatically.
