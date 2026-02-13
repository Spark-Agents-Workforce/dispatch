# Orchestration Patterns — Research for Dispatch Design

**Purpose:** Survey of orchestration models and why the 911 dispatcher pattern wins

---

## Patterns Considered

### 1. Manager Pattern
The orchestrator is a smart manager who can do work when needed and delegates when appropriate.

**Pros:** Flexible, can handle edge cases, feels natural
**Cons:** "When needed" always expands. Manager becomes bottleneck. The very flexibility is the failure mode.

**Verdict:** ❌ Failed in production. A manager who CAN do work WILL do work.

### 2. Router Pattern (Pure)
Stateless request router. Maps input to agent, forwards, done. No intelligence, no context, no memory.

**Pros:** Fast, simple, never does work
**Cons:** Dumb routing leads to bad context assembly. Agent receives "fix it" with no details. Quality of delegation suffers.

**Verdict:** ❌ Too dumb. Routing without context assembly is just forwarding email.

### 3. 911 Dispatcher Pattern ✅
Intelligent routing with zero execution. The dispatcher understands the request, assembles context, routes to the right responder, and tracks status. But NEVER leaves the switchboard.

Key properties:
- **Zero execution** — the dispatcher never goes on the call
- **Intelligent routing** — understands the request well enough to pick the right responder
- **Context assembly** — provides the responder with everything they need
- **Status tracking** — knows what's in flight, reports back
- **Always available** — because they never leave the desk

**Pros:** Combines routing intelligence with zero-work guarantee. The hard boundary prevents scope creep. Always available to the user.
**Cons:** Requires discipline in the SOUL.md to maintain the boundary. Slightly more latency than a pure router (context assembly takes a few seconds).

**Verdict:** ✅ The right pattern. Intelligent enough to be useful, constrained enough to never bottleneck.

### 4. Pub/Sub Pattern
Events published to a bus, agents subscribe to topics they handle. No central orchestrator.

**Pros:** Fully decentralized, scales infinitely, no single point of failure
**Cons:** No one assembles context. No one tracks status. No one answers "what's happening?" User loses visibility.

**Verdict:** ❌ Loses the human-in-the-loop. The user needs a single point of contact.

### 5. Hierarchical Pattern
Multi-level management. Dispatch → Department leads → Individual agents.

**Pros:** Scales to hundreds of agents, natural decomposition
**Cons:** Overkill for small-to-medium fleets. Adds latency. More points of failure.

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
When a coding agent is working on a project and you spawn a new session for a follow-up task, it starts from zero. Doesn't know the codebase decisions, doesn't remember prior implementations, doesn't know what's done. All that context — gone.

When you `sessions_send` to its main session, it wakes up with full context. Reads MEMORY.md, sees yesterday's work, knows the project state. Can pick up exactly where it left off.

**This distinction is the difference between a team of colleagues and a pool of amnesiacs.**

## Parallel Execution Model

The dispatcher's superpower: launching multiple agents simultaneously.

```
User: "Launch prep — images, copy, and tweets"

WRONG (serial):
  t=0:  send to image agent → wait 3 min → result
  t=3:  send to writing agent → wait 3 min → result
  t=6:  send to social agent → wait 2 min → result
  Total: 8 minutes

RIGHT (parallel):
  t=0:  send to all three simultaneously
  t=2:  social agent finishes → report to user
  t=3:  image agent finishes → report to user
  t=3:  writing agent finishes → report to user
  Total: 3 minutes
```

Report results as they arrive. Don't wait for all to finish.

## Dependent Task Chains

Some tasks have dependencies: A must finish before B can start.

```
User: "Design the new logo, then use it on the landing page"

  t=0:  send to image agent for logo
  t=3:  image agent finishes → report to user → send to design agent with logo
  t=6:  design agent finishes → report to user
```

Dispatch tracks the dependency and triggers the next step automatically.
