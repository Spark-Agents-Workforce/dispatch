# Atlas Autopsy — Why the Previous Orchestrator Failed

**Date:** 2026-02-12
**Author:** Architect 🏛️
**Purpose:** Document what went wrong with Atlas so Dispatch doesn't repeat it

---

## What Atlas Was

Atlas 🔱 was the original orchestrator for Josh's 18-agent workforce. It sat in the `main` agent slot and handled all incoming messages from Josh via iMessage, Slack, and webchat. It was supposed to route tasks to specialist agents.

## What Went Wrong

### Failure 1: The Work Trap

Atlas couldn't resist doing work. A "quick" research task? Atlas would do it instead of spawning a subagent. A file that needed reading? Atlas would read it and summarize. A plan that needed writing? Atlas would draft it.

**Impact:** Josh was locked out during every work session. Couldn't send new instructions, couldn't course-correct, couldn't reach other agents. Average lockout: 2-5 minutes per task. Multiple lockouts per session.

**Root cause:** No hard boundary between "routing" and "work." The SOUL.md had soft language: "prefer to route" and "try to delegate." "Prefer" and "try" leave room for exceptions, and exceptions become the rule.

### Failure 2: Context Bloat

Atlas loaded 4-6 files at every session start: SOUL.md, AGENTS.md, USER.md, MEMORY.md, and daily notes. Often 50KB+ of context before Josh said a word. This caused:
- Slow first response (5-10 seconds instead of instant)
- Faster compaction (large context = fewer messages before hitting limits)
- Loss of conversation history during compaction

**Root cause:** Boot sequence loaded everything "just in case" instead of on-demand.

### Failure 3: Named Agents Spawned as Subagents

Atlas frequently used `sessions_spawn` for named agents instead of `sessions_send` to their main session. This meant:
- Mason would start Agent Royale tasks with zero memory of prior work
- Printer would lose position tracking context
- Twin would lose tweet thread continuity
- Every agent effectively had amnesia between tasks

**Root cause:** No clear documentation of the `sessions_send` vs `sessions_spawn` distinction. The routing table didn't specify dispatch method per agent.

### Failure 4: Serial Execution

When Josh requested multi-agent work, Atlas would spawn one agent, wait for completion, then spawn the next. A 3-agent task that could finish in 3 minutes took 9 minutes.

**Root cause:** No explicit "parallel by default" principle. Atlas defaulted to sequential because it felt safer.

### Failure 5: Delayed Reporting

Atlas would wait for all tasks to complete before reporting back to Josh. Even when one agent finished in 30 seconds, Josh wouldn't know until the slowest agent finished.

**Root cause:** Batch mindset instead of streaming mindset.

### Failure 6: Ambiguity Paralysis

When a request was ambiguous, Atlas would spend 30-60 seconds trying to figure out the right interpretation instead of asking Josh a quick clarifying question.

**Root cause:** Atlas was optimized for "look smart" instead of "be fast."

## What Atlas Got Right

Not everything was bad:
- **Routing table concept** — having a documented mapping of tasks to agents was correct
- **Acknowledgment pattern** — confirming receipt before routing was good UX
- **In-flight tracking** — maintaining awareness of active tasks was valuable
- **Context assembly** — providing agents with rich context improved their output quality

## Lessons for Dispatch

1. **Hard boundary, not soft preference.** "NEVER do work" not "prefer to delegate."
2. **2-file boot, not 6-file boot.** SOUL.md + AGENTS.md. Everything else on demand.
3. **sessions_send for named agents, sessions_spawn for disposable.** Documented per-agent.
4. **Parallel by default.** Explicit principle.
5. **Stream results.** Report as each agent finishes, don't batch.
6. **Ask fast, don't guess slow.** One clarifying question beats 60 seconds of wrong guessing.
7. **30-second ceiling.** If response takes longer, you're doing work. Stop.
