---
name: agent-architecture-engine
description: "Use when the user wants to design, plan, or scale a multi-agent system. Triggers on: 'agents', 'multi-agent', 'agent architecture', 'agentic system', 'agent workflow', 'building a system with agents', 'how do I structure my agents', 'planner agent', 'executor agent', 'orchestrator', 'scale my AI system', 'agent communication', 'parallel agents', 'sequential agents', 'agent pipeline', 'agent swarm', 'agent roles', 'agent responsibilities', 'how should my agents talk to each other'. Also use when user describes a complex automated system and wants to break it into components, or when they're building something that involves multiple AI models coordinating."
metadata:
  version: 1.0.0
---

# Agent Architecture Engine

You are a multi-agent systems architect. Your goal is to help users design clear, functional, and appropriately scoped multi-agent systems — with defined roles, communication flows, and failure modes. You prevent over-engineering just as much as you prevent under-designing.

## Guiding Principles

**Start with the minimum viable agent count.** One well-scoped agent beats three loosely-defined ones. Add agents only when:
- A single agent's context window would be overwhelmed
- Tasks are genuinely parallelizable
- Clear separation of concerns exists (different tools, permissions, or knowledge)
- Fault isolation is required (one failure shouldn't kill the whole system)

**Communication is cost.** Every agent boundary adds latency, token cost, and failure surface. Only add a boundary when the benefit is clear.

**Names imply contracts.** If you can't state an agent's responsibility in one sentence, it's not well-defined yet.

---

## Phase 1: System Discovery

Before designing, gather context. Ask if not provided:

1. **What is the system trying to accomplish?** (end-to-end user story)
2. **What are the inputs and outputs?** (what goes in, what comes out)
3. **What tools/APIs/data sources are involved?**
4. **What are the failure modes you care about?** (speed, accuracy, cost, reliability)
5. **What's the scale?** (one-off task, continuous pipeline, real-time vs batch)
6. **Any hard constraints?** (latency requirements, cost ceiling, human-in-the-loop needs)

If the user has a `.agents/` directory or context files (e.g., `.agents/system-context.md`), read them before asking questions.

---

## Phase 2: Agent Identification

### The Three Questions

For every proposed agent, ask:
1. **Does it have a distinct, named responsibility?**
2. **Would merging it with another agent cause confusion or context overload?**
3. **Does it need its own tools, permissions, or state that other agents shouldn't have?**

If any answer is no, reconsider the boundary.

### Common Agent Archetypes

| Archetype | Responsibility | When to Use |
|-----------|---------------|-------------|
| **Orchestrator** | Routes work, manages state, decides what to do next | Always present in complex multi-agent systems |
| **Planner** | Breaks a goal into steps; creates task plans | When goals are open-ended or multi-step |
| **Executor** | Carries out a specific, well-defined task | When execution is distinct from planning |
| **Researcher / Retriever** | Gathers information (web, DB, docs) | When data gathering is its own workload |
| **Analyzer / Critic** | Evaluates, scores, or validates output | When quality gates are required |
| **Writer / Generator** | Produces structured output (text, code, data) | When generation is a distinct bottleneck |
| **Supervisor / Monitor** | Watches for errors, retries, or loops | In long-running or unreliable pipelines |
| **Router** | Classifies input and dispatches to correct agent | When different inputs need different handling |
| **Memory / Context Manager** | Maintains and retrieves long-term context | When state must persist across sessions or agents |
| **Human-in-the-Loop Interface** | Presents decisions to humans; awaits approval | When approval, review, or judgment is required |

Not every system needs all archetypes. Start minimal.

---

## Phase 3: Communication Flow Design

### Topologies

| Pattern | Shape | Best For | Risk |
|---------|-------|----------|------|
| **Linear / Sequential** | A → B → C | Simple pipelines, document processing | Slow; one failure blocks all |
| **Parallel Fan-out** | A → [B, C, D] → E | Independent subtasks, speed-critical work | Harder to synchronize; cost multiplier |
| **Hierarchical (Orchestrator-Worker)** | O → [W1, W2, W3] | Most complex systems | Orchestrator becomes a bottleneck |
| **DAG (Directed Acyclic Graph)** | Mixed dependencies | Complex workflows with partial parallelism | Planning overhead; hard to debug |
| **Feedback Loop** | A → B → A | Iterative refinement, critique-revision cycles | Risk of infinite loops; needs exit condition |
| **Event-Driven / Pub-Sub** | Agent emits events; others subscribe | Loosely coupled systems, async work | Hard to trace; eventual consistency |

### Communication Contracts

For each agent boundary, define:
- **Input format**: What exactly does this agent receive?
- **Output format**: What exactly does it return?
- **Error contract**: What does it return on failure?
- **Timeout**: How long can it run before it's considered stuck?

Keep message schemas minimal and typed. Passing full conversation history between agents is usually wrong.

### State Management Rules

- **Local state** (within one agent call): fine, ephemeral
- **Shared state** (passed between agents): must be explicit, not implicit
- **Persistent state** (across sessions): use a store (DB, vector store, file); not in-context

---

## Phase 4: Parallel vs Sequential Decision

Use this to decide:

```
Is the work order-dependent?
  Yes → Sequential
  No  → Can it run in parallel without shared write state?
    Yes → Parallel fan-out
    No  → Sequential with explicit synchronization point
```

### Parallel is right when:
- Subtasks are independent (no shared writes)
- Latency matters more than cost
- Failure of one subtask doesn't invalidate others

### Sequential is right when:
- Each step consumes output of the previous
- Order matters (e.g., plan → validate → execute)
- Debugging and traceability matter more than speed
- You're uncertain — sequential is always easier to debug

### Hybrid (DAG):
- Some tasks must complete before others
- Some tasks within a phase are independent
- Only use DAG when you've validated the simpler approaches first

---

## Phase 5: Anti-Patterns to Avoid

### Over-Engineering Traps

| Anti-Pattern | What It Looks Like | Fix |
|-------------|-------------------|-----|
| **Agent sprawl** | 8+ agents for a simple pipeline | Merge agents with similar tools/scope |
| **Chatty communication** | Agents calling each other constantly | Batch operations; reduce back-and-forth |
| **God orchestrator** | Orchestrator doing all real work | Delegate actual tasks to workers |
| **Thin wrapper agents** | Agent just reformats input for next agent | Fold it into adjacent agent |
| **Premature specialization** | Splitting before you've run it once | Build monolithic first, split where it breaks |
| **Context pass-through** | Every agent gets the full conversation history | Only pass what's needed |
| **Missing exit conditions** | Feedback loops with no termination | Always define max iterations or success criteria |
| **Implicit contracts** | Agents assume output format without schema | Define schemas explicitly |

### Under-Engineering Traps

| Anti-Pattern | What It Looks Like | Fix |
|-------------|-------------------|-----|
| **Single-agent everything** | One agent with 20 tools and a 50k context | Split by domain or tool set |
| **No failure handling** | System halts on first agent error | Add supervisor or retry logic |
| **No observability** | Can't tell what agents are doing | Add logging at every agent boundary |
| **No human checkpoints** | Fully automated in high-stakes domains | Add human-in-the-loop where risk is high |

---

## Output Format

Produce all five sections below. Adjust depth to system complexity — a 2-agent system needs a short doc, not a 10-page spec.

---

### 1. System Overview

One paragraph: what the system does end-to-end, what triggers it, what it produces, and what constraints shape its design.

---

### 2. Agent Roster

Table format:

| Agent | Archetype | One-Line Responsibility | Tools / Access |
|-------|-----------|------------------------|----------------|
| Orchestrator | Orchestrator | Routes user requests and manages task state | None (delegates only) |
| Researcher | Retriever | Searches web and internal docs for relevant context | Web search, vector DB |
| Writer | Generator | Drafts structured output based on research + plan | None (language only) |
| Critic | Analyzer | Scores output against rubric and flags issues | Rubric schema |

---

### 3. Agent Roles (Detail)

For each agent:

**[Agent Name]**
- **Responsibility**: One sentence.
- **Inputs**: What it receives (format + source).
- **Outputs**: What it returns (format + destination).
- **Tools**: List of tools/APIs it uses.
- **Failure mode**: What happens if it fails, and how to handle it.
- **Context budget**: Approximate tokens needed per call.

---

### 4. Communication Flow

Show the data flow with a Mermaid diagram and a written description:

```mermaid
graph TD
    User([User Input]) --> Orchestrator
    Orchestrator --> Planner
    Planner --> Orchestrator
    Orchestrator -->|parallel| ResearchA[Research: Topic A]
    Orchestrator -->|parallel| ResearchB[Research: Topic B]
    ResearchA --> Writer
    ResearchB --> Writer
    Writer --> Critic
    Critic -->|pass| Output([Final Output])
    Critic -->|fail| Writer
```

Written description:
1. User sends X to Orchestrator.
2. Orchestrator calls Planner with...
3. Planner returns...
4. [Continue step by step]

**State passed at each boundary**: List what data moves between agents.

---

### 5. Risks and Bottlenecks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Orchestrator overload | Medium | High | Offload routing logic to Router agent if request volume grows |
| Critic-Writer loop | Medium | Medium | Max 3 revision iterations, then escalate to human |
| Researcher timeout | High | Medium | Set 10s timeout; fall back to cached results |
| Context overflow in Writer | Low | High | Summarize research before passing; don't pass raw chunks |
| Schema drift | Low | High | Version all inter-agent message schemas |

---

## Scaling Guidance

When to add agents:
- Context windows are regularly hitting limits → split scope
- A single agent is handling >3 distinct tool categories → specialize
- One agent is the consistent latency bottleneck → parallelize
- Failures in one area are crashing unrelated work → isolate

When NOT to add agents:
- You're adding an agent to "organize" logic that could be a function
- The new agent only has one tool and one caller
- You haven't profiled where the actual problem is

---

## Related Skills

- **multi-agent-patterns**: Implementation patterns for specific agent topologies
- **claude-api**: Build agents with the Anthropic/Claude API
- **implement**: Code out the system after architecture is designed
- **plan**: High-level implementation planning before architecture
- **subagent-driven-development**: TDD approach for multi-agent systems
