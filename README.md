# AI Engineering Roadmap (Software Engineer Perspective)

## Philosophy
- Goal: build *with* AI reliably, not build AI itself
- Learn concepts before tools — tools change, concepts don't
- Go deep enough to understand failure modes, not just happy paths

---

## Phase 1: Core Concepts (Foundation)

Learn in this order — each builds on the previous.

### 1. Agent
What it is: a model in a loop (perceive → think → act → repeat)

- Read: [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — Anthropic, ~30 min
- Read: [Basic agent workflow patterns](https://platform.claude.com/cookbook/patterns-agents-basic-workflows) — chaining, parallelization, routing ✓
- Goal: understand the loop, understand why agents fail (hallucination, tool errors, infinite loops)

### 2. MCP (Model Context Protocol)
What it is: standard protocol for connecting agents to external tools

- Read: MCP spec at modelcontextprotocol.io
- Do: read a minimal MCP server implementation (~100 lines, TypeScript or Python SDK)
- Do: wire a simple MCP server into Claude Code yourself
- Goal: understand the handshake — how a model discovers tools, calls them, handles results

### 3. Harness
What it is: the scaffolding that runs agents (lifecycle, permissions, memory, retries)

- Observe: Claude Code *is* a harness — use it as your reference implementation
- Read: `.claude/settings.json` structure, how hooks work, how skills are registered
- Goal: understand what you'd have to build yourself if the harness didn't exist

### 4. Skills
What it is: reusable, named behaviors packaged for the harness to invoke

- Observe: skills available in Claude Code (`/help`)
- Do: read an existing skill implementation
- Goal: understand skills as harness-level abstractions, distinct from MCP tools

---

## Phase 2: Tool Reliability

The gap between a demo agent and a production agent is mostly reliability.

### Key concepts
- **Idempotency**: tool calls that are safe to retry
- **Structured outputs**: reduce parsing failures by forcing schema-valid responses
- **Retry logic**: when to retry, when to fail fast, exponential backoff
- **Circuit breakers**: don't hammer a failing downstream service
- **Tool design**: clear names, single responsibility, explicit error contracts

### Resources
- Anthropic tool use docs (error handling section)
- Read production postmortems from teams shipping agents (search Substack, eng blogs)
- Study how Claude Code handles tool permission failures and retries in practice

### Things to build
- A tool that fails gracefully and tells the agent *why* (not just "error")
- A tool with retry + idempotency key
- An agent that recovers from a tool failure without human intervention

---

## Phase 3: Evaluation (Evals)

The difference between "it seems to work" and "I know it works."

### Why evals matter
Without evals, every change is a gamble. Evals let you improve the system without breaking it.

### Types of evals
| Type | What it tests | When to use |
|---|---|---|
| Unit eval | Single prompt → expected output | Fast iteration on a specific behavior |
| Integration eval | Full agent run → outcome | End-to-end correctness |
| LLM-as-judge | Model grades model output | Subjective quality at scale |
| Trajectory eval | Was each step in the agent loop correct? | Agent-specific, catches mid-run failures |

### Resources
- **[promptfoo](https://promptfoo.dev)** — open source, practical, start here
- **Hamel Husain's writing on evals** — clearest thinking on this topic publicly available
- Anthropic's eval guidance in their docs
- RAGAS (for RAG-specific evals, if relevant later)

### Things to build
- An eval suite for a tool you built in Phase 2
- An LLM-as-judge eval for a subjective output
- A regression test that catches a prompt change breaking behavior

---

## Phase 4: Systems Thinking

Putting it together at production scale.

- Observability: logging agent traces, tool call latency, failure rates
- Cost management: caching (prompt caching), batching, model routing by task complexity
- Human-in-the-loop: when to pause and ask vs proceed autonomously
- Security: prompt injection, tool permission scoping, output sanitization

---

## Career Context

- Core LLM development is consolidating at a few labs — not the growth path for most engineers
- Application layer (agents, reliability, evals) is where most value is created and where talent is scarce
- Durable edge: understanding *why* systems fail, not just how to wire them up
- At Google: production-scale AI problems are better training ground than most places

---

## Current Progress

- [x] Understood agent/MCP/harness/skill conceptually
- [ ] Read "Building effective agents" (Anthropic) — up next
- [x] Read basic workflow patterns cookbook
- [ ] Read MCP spec
- [ ] Build a minimal MCP server
- [ ] Wire MCP server into Claude Code
- [ ] Build a tool with retry + idempotency
- [ ] Set up promptfoo, write first eval suite
