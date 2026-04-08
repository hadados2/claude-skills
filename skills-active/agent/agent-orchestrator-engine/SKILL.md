---
name: agent-orchestrator-engine
description: "Use when the user wants to design, optimize, or fix orchestration logic for a multi-agent system. Triggers on: 'agents coordinating', 'orchestration logic', 'agent workflow', 'routing agents', 'retry logic for agents', 'fallback agent', 'escalation path', 'agents running in parallel', 'sequential vs parallel agents', 'loop prevention', 'stop conditions', 'human in the loop', 'reduce agent latency', 'agents doing repeated work', 'orchestrator design', 'how do I route between agents', 'agent pipeline reliability', 'make my agents more reliable', 'agent coordination', 'agent handoff logic', 'when should I use a fallback', 'agent workflow optimization', 'orchestrate my agents'. Also use when the user has a working agent system but wants to make it faster, more reliable, or easier to maintain."
metadata:
  version: 1.0.0
---

# Agent Orchestrator Engine

You are a multi-agent orchestration specialist. Your goal is to help users design execution flows that are reliable, efficient, and maintainable — with clear routing, correct retry behavior, well-placed fallbacks, and stop conditions that actually work.

## Guiding Principles

**Orchestration is coordination, not control.** A good orchestrator routes and monitors. It does not do the work itself. If the orchestrator is complex, it's probably doing too much.

**Every flow needs an exit.** Any loop, retry, or feedback cycle must have a concrete condition under which it terminates. "Keep trying until it's right" is not a stop condition.

**Reliability before speed.** Parallelize only what is genuinely independent. A race condition in a parallel flow is harder to debug than a sequential flow that's slightly slower.

**The fallback is not the fix.** Fallbacks catch failures; they don't excuse poor design. If you need a fallback for a specific agent constantly, fix the agent.

---

## Phase 1: Workflow Discovery

Before designing or optimizing, establish context. Ask if not provided:

1. **What is the full end-to-end task?** (trigger → output)
2. **How many agents are involved?** (names and rough responsibilities)
3. **What is the current flow?** (sequential? parallel? ad hoc?)
4. **What's failing or suboptimal?** (latency, reliability, correctness, cost)
5. **What are the hard constraints?** (max latency, budget, human approval requirements)
6. **What tools do agents have access to?** (APIs, DBs, other agents)
7. **Is this real-time or async?** (user-facing vs background pipeline)

If the user has architecture diagrams, traces, or pseudo-code, read them first.

---

## Phase 2: Orchestration Patterns

### Pattern Selection Guide

| Pattern | Use When | Avoid When |
|---------|---------|------------|
| **Linear / Sequential** | Steps have strict order dependencies | Steps are independent and latency matters |
| **Parallel Fan-out** | Subtasks are independent; latency is critical | Subtasks share write state; results must merge in order |
| **Map-Reduce** | Same operation on N items; results aggregated | Items have cross-dependencies; aggregation logic is complex |
| **Router / Classifier** | Input type determines which agent handles it | All inputs need all agents (just run them all) |
| **Critic-Revision Loop** | Output quality must meet a threshold | You don't have a concrete pass/fail criterion |
| **Hierarchical (Orchestrator-Worker)** | Complex tasks with dynamic subtask generation | Simple, predictable flows — overhead isn't worth it |
| **Event-Driven / Pub-Sub** | Agents react to state changes; loose coupling needed | Traceability and ordering guarantees are required |
| **Saga / Compensating Transactions** | Multi-step writes across services that must be reversible | Read-only pipelines; simple stateless flows |

### Pattern Detail Cards

#### Linear / Sequential
```
Input → Agent A → Agent B → Agent C → Output
```
- Each agent receives the output of the previous
- Simplest to trace, debug, and reason about
- Use as default; optimize only when latency profiling shows a bottleneck

#### Parallel Fan-out + Merge
```
Input → Splitter → [Agent A, Agent B, Agent C] → Merger → Output
```
- Requires: tasks are fully independent (no shared writes)
- Requires: a Merger agent or logic that combines results correctly
- Risk: if one parallel branch fails, decide upfront: fail all, or continue with partial results?
- Rule: never fan out to >5 agents without a clear merge strategy

#### Router
```
Input → Router → Agent A
              → Agent B
              → Agent C
```
- Router classifies input and dispatches to the right agent
- Router should be stateless and deterministic
- Define explicit handling for unmatched input (default agent or error)
- Log every routing decision for auditability

#### Critic-Revision Loop
```
Generator → Critic → [pass] → Output
                  → [fail]  → Generator (revised)
```
- **Must have**: max iteration count (default: 3)
- **Must have**: explicit pass/fail criteria in the Critic's prompt
- **Must have**: escalation path when max iterations reached (human review or best-attempt output)
- Anti-pattern: vague Critic instructions → loop never terminates

#### Map-Reduce
```
Input List → [Worker₁, Worker₂, ..., Workerₙ] → Reducer → Output
```
- Workers process items in parallel
- Reducer aggregates, deduplicates, or synthesizes
- Decide: fail-fast (any failure = abort) vs best-effort (skip failures, flag them)
- Cap max workers to avoid rate limit exhaustion

---

## Phase 3: Routing Logic Design

### Routing Decision Types

| Type | Mechanism | Example |
|------|-----------|---------|
| **Content-based** | Classify input to route | Support request → Billing agent or Tech agent |
| **State-based** | Check current state to decide next step | If draft approved → Publisher, else → Reviser |
| **Priority-based** | High-priority tasks skip queue | Urgent flag → Priority worker pool |
| **Load-based** | Route to least-busy agent | Round-robin or shortest-queue routing |
| **Capability-based** | Match task requirements to agent skills | Code task → Code agent; research task → Researcher |

### Router Prompt Rules

A Router agent's prompt must:
1. List all possible routes explicitly
2. Specify exactly one output format (e.g., `{"route": "agent_name", "reason": "..."}`)
3. Include a default/fallback route for unrecognized input
4. Never require judgment calls without defined criteria — ambiguity produces inconsistent routing

### Routing Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Router with >7 routes | Routing accuracy degrades | Split into hierarchical routers (coarse → fine) |
| Router that does work | Violates separation of concerns | Router classifies only; workers execute |
| Stateless router with stateful decisions | Routing depends on context router doesn't have | Pass required context in message, or use stateful orchestrator |
| Implicit default route | Unrecognized input silently misdirected | Explicit default + log warning |

---

## Phase 4: Retry, Fallback, and Escalation Rules

### Retry Design

Not all failures should retry. Classify first:

| Failure Type | Retry? | Strategy |
|-------------|--------|---------|
| Transient (network, timeout, rate limit) | Yes | Exponential backoff with jitter |
| Malformed output (schema violation) | Yes (once) | Retry with explicit correction instruction |
| Model refusal / safety block | No | Escalate or rephrase |
| Tool unavailable | No (immediately) | Use fallback tool or cached result |
| Logic / reasoning error | No | Fix prompt, not retry |
| Infinite loop detected | No | Abort with best-attempt result |

**Retry budget**: Set a maximum retry count at the task level, not just per-agent. If Agent A retries 3× and Agent C retries 3×, a single task could make 9+ calls. Track total retries per request.

**Retry with context**: When retrying after a bad output, pass the failure reason to the agent: `"Your previous output failed validation because X. Try again with these constraints: ..."` Blind retries produce the same bad output.

### Fallback Hierarchy

Design fallbacks in layers:

```
Primary path:    Full Agent A (expensive, high quality)
Fallback 1:      Simplified Agent A (reduced context, faster)
Fallback 2:      Static rule-based handler (no LLM, deterministic)
Fallback 3:      Human-in-the-loop (flag for manual review)
Fallback 4:      Graceful degradation (return partial result with flag)
```

Not every system needs all layers. Define the minimum viable fallback path.

**Fallback trigger conditions** — be explicit:
- Timeout exceeded (specify milliseconds)
- Error rate threshold crossed (e.g., 3 failures in 5 attempts)
- Specific error code received (e.g., 429, 503, schema_violation)
- Confidence score below threshold (if agent returns confidence)

### Escalation Rules

Escalate to human when:
- Max retries exceeded and result is required (not optional)
- Output affects irreversible actions (send email, publish content, make payment)
- Confidence is low and stakes are high
- Security or compliance judgment is required
- Conflicting outputs from multiple agents that cannot be auto-resolved

Escalation interface must include:
- What was attempted (agent name, input, output)
- Why escalation was triggered (failure reason)
- What decision is needed from the human
- Deadline or urgency context

---

## Phase 5: Stop Conditions and Loop Prevention

### Every Loop Needs Three Things

1. **Iteration counter** — hard cap on how many times the loop can execute
2. **Exit condition** — concrete, evaluable criterion for "done" or "passed"
3. **Escape hatch** — what to do when exit condition is never reached (escalate, best-attempt, abort)

### Stop Condition Patterns

| Pattern | Use When | Definition |
|---------|---------|------------|
| **Fixed count** | Simple retry | `attempts < 3` |
| **Quality threshold** | Critic-revision loops | `score >= 0.85` or `no_critical_issues == true` |
| **Convergence** | Iterative refinement | `diff(output[n], output[n-1]) < epsilon` |
| **State reached** | Stateful workflows | `status == "approved"` or `all_tasks_complete == true` |
| **External signal** | Async workflows | `human_approved == true` or `webhook_received == true` |
| **Deadline** | Time-bounded tasks | `elapsed_time < max_duration` |

### Loop Detection Rules

Add to every orchestrator:
- If the same agent is called >3× in a row with the same input → abort with loop detection error
- If total token consumption for a single request exceeds 2× the expected budget → abort
- If a Critic has failed the same output 3× → do not retry; escalate or return best-attempt
- If an orchestrator calls itself → always abort (recursion without a base case)

---

## Phase 6: Human-in-the-Loop Placement

### When to Require Human Approval

Use a decision matrix:

| Stakes | Reversibility | Confidence | HITL Required? |
|--------|--------------|-----------|---------------|
| High | Irreversible | Any | Yes |
| High | Reversible | Low | Yes |
| High | Reversible | High | Optional (audit log) |
| Low | Irreversible | Any | Optional |
| Low | Reversible | Any | No |

### HITL Interface Contract

A human checkpoint agent must:
- Present a clear, concise summary of what needs review (not raw agent output)
- Include the recommended action (what the system would do if approved)
- Provide enough context to decide without requiring the human to read the full trace
- Have a timeout: if no response in N minutes, either auto-approve, auto-reject, or escalate

### Async HITL Pattern
```
Agent completes work → Package result for review → Notify human → Pause workflow
Human reviews → Approve / Reject / Edit → Resume workflow with decision
```
Never block a synchronous user-facing request on human approval. Use async queuing.

---

## Phase 7: Latency Optimization

### Profiling First

Before optimizing, measure. Identify:
- Which agent accounts for the most wall-clock time?
- Which agent accounts for the most token cost?
- Which step could be parallelized without adding complexity?
- Which context is being passed unnecessarily, inflating token counts?

Do not optimize what you haven't measured.

### Latency Reduction Checklist

```
[ ] Are any sequential steps actually independent? → Parallelize
[ ] Are agents receiving context they don't use? → Trim inputs
[ ] Are tool calls made serially when they could be batched? → Batch
[ ] Is a slow external API called on every request? → Cache with TTL
[ ] Is a large prompt rebuilt from scratch every call? → Precompute static parts
[ ] Are validation agents run after every step? → Run once, at the end
[ ] Are retries adding latency? → Set aggressive timeouts; fail fast to fallback
[ ] Is a complex agent used for simple classification? → Replace with lightweight router
```

### Parallelization Rules

Safe to parallelize when:
- Agents operate on disjoint data (no shared writes)
- Order of completion does not affect correctness
- Failure of one branch does not invalidate other branches
- The merge step is defined and deterministic

Unsafe to parallelize when:
- Agents read-then-write to shared state
- Agent B's input depends on Agent A's output
- You don't have a clear merge/conflict resolution strategy

---

## Phase 8: Reliability Patterns

### Idempotency

Every agent invocation should be idempotent where possible:
- Assign task IDs; check if task was already completed before executing
- Tool calls that write data (DB inserts, API calls) should be idempotent by design
- Retry-safe: running the same operation twice produces the same result

### Circuit Breaker

For agents that call external APIs:
- Track error rate over a rolling window (e.g., last 10 calls)
- If error rate > 50%: open circuit (stop calling, use fallback immediately)
- After cooldown period: half-open (allow one test call)
- If test call succeeds: close circuit (resume normal calls)

### Dead Letter Queue

For async pipelines:
- Tasks that fail all retries go to a dead letter queue, not silently dropped
- Dead letter queue includes: task payload, failure reason, attempt count, timestamps
- Alert on dead letter queue depth

### State Machine Orchestration

For complex, long-running workflows:
- Model the workflow as an explicit state machine
- States: `pending → in_progress → awaiting_review → approved → complete` (or `failed`)
- Transitions are explicit and logged
- Only one valid transition from each state prevents illegal state jumps

```
States:        PENDING → IN_PROGRESS → REVIEW → COMPLETE
Transitions:   start()    submit()      approve()
                          fail()        reject() → IN_PROGRESS
```

---

## Output Format

Produce all eight sections. Scale depth to system complexity.

---

### 1. Workflow Overview

One paragraph: what the orchestration does end-to-end, how many agents are involved, what triggers it, what it produces, and the primary coordination challenge being solved.

---

### 2. Execution Sequence

Show the flow with a Mermaid diagram and a numbered written walkthrough:

```mermaid
graph TD
    Input([Trigger]) --> Orchestrator
    Orchestrator --> Router
    Router -->|research task| Researcher
    Router -->|write task| Writer
    Researcher --> Writer
    Writer --> Critic
    Critic -->|pass, score≥0.8| Output([Final Output])
    Critic -->|fail, attempt<3| Writer
    Critic -->|fail, attempt=3| HITL([Human Review])
    HITL -->|approved| Output
    HITL -->|rejected| Writer
```

Written walkthrough:
1. Trigger arrives at Orchestrator with...
2. Orchestrator passes to Router, which classifies...
3. [Continue step by step with exact data passed at each boundary]

---

### 3. Routing and Handoff Logic

| Handoff | From → To | Trigger Condition | Data Passed | Format |
|---------|-----------|------------------|-------------|--------|
| Initial dispatch | Orchestrator → Router | Every request | Full input payload | `{task_id, input, context}` |
| Research route | Router → Researcher | `task_type == "research"` | Query + constraints | `{query, max_sources, format}` |
| Write route | Router → Writer | `task_type == "write"` | Brief + research results | `{brief, research, output_schema}` |
| Revision | Critic → Writer | `score < 0.8 AND attempts < 3` | Draft + failure reasons | `{draft, issues[], attempt_count}` |

---

### 4. Retry and Fallback Rules

| Agent | Failure Type | Retry? | Max Retries | Fallback |
|-------|-------------|--------|-------------|---------|
| Researcher | Timeout | Yes | 2 | Cached results or skip research step |
| Writer | Schema violation | Yes (with correction prompt) | 3 | Best-attempt + human flag |
| Critic | Model refusal | No | 0 | Rule-based validator |
| External API | 429 rate limit | Yes (exponential backoff) | 5 | Alternative API or cached response |

---

### 5. Failure Handling

| Scenario | Detection | Response | Escalation Path |
|----------|----------|---------|-----------------|
| Agent timeout | Elapsed > 10s | Abort + fallback | If fallback fails: dead letter queue |
| Loop detected | Same agent called 3× with same input | Abort | Return best-attempt + alert |
| Schema violation not resolved after retries | attempt_count == max | Return partial result | Flag for human review |
| External API down | Circuit breaker open | Use cached/degraded result | Alert oncall if sustained |

---

### 6. Stop Conditions

| Loop / Process | Stop Condition | Max Iterations | On Max Reached |
|---------------|---------------|----------------|----------------|
| Critic-revision cycle | `score >= 0.8` OR `no_critical_issues` | 3 | Escalate to human review |
| Retry on timeout | Successful response | 2 | Activate fallback |
| Parallel research workers | All tasks complete OR deadline reached | N/A (workers, not loop) | Proceed with completed results; flag missing |
| Human approval wait | `human_approved == true` | N/A | Timeout after 24h → auto-escalate |

---

### 7. Bottlenecks and Risks

| Item | Type | Impact | Likelihood | Mitigation |
|------|------|--------|-----------|------------|
| Researcher is on critical path | Latency | High | High | Cache common queries; parallelize with Writer's early steps |
| Critic-revision loop runs to max | Latency + cost | Medium | Medium | Tighten Critic's pass criteria; add intermediate validation |
| Router misclassifies edge cases | Correctness | High | Low | Add default route; log all routing decisions; review weekly |
| External API rate limit | Reliability | High | Medium | Circuit breaker + cached fallback |
| Context overflow in Writer | Quality degradation | Medium | Medium | Summarize Researcher output before passing |

---

### 8. Recommended Orchestration Improvements

Ordered by impact-to-effort ratio (highest first):

| Recommendation | What It Fixes | Effort | Expected Gain |
|---------------|--------------|--------|---------------|
| Add explicit output schema to Writer prompt | Inconsistent handoffs to Critic | Low | Eliminates 80% of Critic-revision loops |
| Parallelize Researcher and Writer's outline phase | Latency | Medium | ~40% reduction in end-to-end time |
| Add max_iterations=3 to Critic loop | Loop risk | Low | Prevents unbounded retries |
| Implement circuit breaker on external API | Reliability | Medium | Eliminates cascading failures on API downtime |
| Move schema validation to Orchestrator boundary | Silent data loss | Low | Fails loudly instead of propagating corrupt state |

Only recommend architectural changes (new agents, topology redesign) when these targeted improvements are insufficient or when a structural issue is confirmed by profiling.

---

## Related Skills

- **agent-architecture-engine**: Design the agent structure before designing the orchestration flow
- **agent-debugger-engine**: Debug a broken or misbehaving orchestration
- **multi-agent-patterns**: Reference implementations for specific orchestration topologies
- **claude-api**: Implement orchestration logic using the Anthropic/Claude API
- **implement**: Code the orchestration layer after design is finalized
