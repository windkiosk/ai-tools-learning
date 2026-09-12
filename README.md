# AI Engineering Roadmap (Software Engineer Perspective)

## Philosophy
- Goal: build *with* AI reliably, not build AI itself
- Learn concepts before tools — tools change, concepts don't
- Go deep enough to understand failure modes, not just happy paths
- Read: [My AI Adoption Journey](https://mitchellh.com/writing/my-ai-adoption-journey) (Mitchell Hashimoto) — a high-signal case study on building leverage.

---

## Phase 1: Core Concepts (Foundation)

Learn in this order — each builds on the previous. This phase is the vocabulary for AI infra/agent engineering; model-internals theory (e.g. Transformer architecture) is out of scope — not required to build, harness, or operate agents.

### 1. Agent
What it is: a model in a loop (perceive → think → act → repeat)

- Read: [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) — how models interleave reasoning and action.
- Read: [Plan-and-Solve Prompting](https://arxiv.org/abs/2305.04091) — why devising a plan first improves reasoning.
- Read: [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — Anthropic, ~30 min
- Read: [Basic agent workflow patterns](https://platform.claude.com/cookbook/patterns-agents-basic-workflows) — chaining, parallelization, routing ✓
- Explore: [Plan-and-Execute Agents (LangChain)](https://python.langchain.com/v0.1/docs/modules/agents/agent_types/plan_and_execute/) and [LangGraph Tutorial](https://langchain-ai.github.io/langgraph/tutorials/plan-and-execute/plan-and-execute/).
- Goal: understand the loop, understand why agents fail (hallucination, tool errors, infinite loops), and when to use planning vs. reactive patterns.

### 2. MCP (Model Context Protocol)
What it is: standard protocol for connecting agents to external tools

- Read: MCP spec at modelcontextprotocol.io
- Do: read a minimal MCP server implementation (~100 lines, TypeScript or Python SDK)
- Do: wire a simple MCP server into Claude Code yourself
- Goal: understand the handshake — how a model discovers tools, calls them, handles results

### 3. Harness
What it is: the scaffolding that runs agents (lifecycle, permissions, memory, retries) and the **environment** they inhabit.

- Read: [Harness Engineering](https://openai.com/index/harness-engineering/) (OpenAI) — why the "harness" is the primary product in an agentic world.
- Read: [Harness Engineering Memo](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering-memo.html) (Martin Fowler) — practical takeaways for software engineers.
- Read: [The Anatomy of an Agent Harness](https://www.langchain.com/blog/the-anatomy-of-an-agent-harness) (LangChain) — architectural breakdown.
- Read: [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) (Anthropic)
- Read: [Harness Design for Long-Running Apps](https://www.anthropic.com/engineering/harness-design-long-running-apps) (Anthropic)
- Read: [Managed Agents](https://www.anthropic.com/engineering/managed-agents) (Anthropic) — decouples the model from execution environment/session storage via stable interfaces, so the harness can evolve without re-encoding assumptions about what Claude can't do.
- Read: [Claude Managed Agents overview](https://platform.claude.com/docs/en/managed-agents/overview) (Anthropic docs) — the product surface for the above: agent/environment/session/event model, hosted vs. self-hosted sandboxes.
- **Deep Dive**: [Session 1: Harness Engineering](./sessions/01_harness_engineering.md) — breakdown of practical implementation.
- Observe: Claude Code *is* a harness — use it as your reference implementation
- Read: `.claude/settings.json` structure, how hooks work, how skills are registered
- Key Practices:
    - **Legibility**: Move design docs and "core beliefs" into the repo as Markdown so agents can reason about them.
    - **Observability**: Give agents access to logs (LogQL) and metrics (PromQL) so they can debug autonomously.
    - **Rigid Architecture**: Use strict patterns and linters to prevent AI-generated "slop" and architectural drift.
- Goal: understand what you'd have to build yourself if the harness didn't exist

### 4. Skills
What it is: reusable, named behaviors packaged for the harness to invoke

- Observe: skills available in Claude Code (`/help`)
- Do: read an existing skill implementation
- Goal: understand skills as harness-level abstractions, distinct from MCP tools

### 5. Context Engineering & Retrieval
What it is: how you get the *right* information into a finite context window — the dominant lever for both agent reliability and cost.

- Read: [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Anthropic; context as a finite, precious resource, and strategies (compaction, tool-result clearing, memory) for managing it across a long agent run.
- Read: [Contextual Retrieval](https://www.anthropic.com/engineering/contextual-retrieval) — Anthropic; the standard fix for the "lost context" problem when chunking documents for RAG (reduces failed retrievals significantly, per their benchmarks).
- Read: [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401) (Lewis et al., 2020) — the paper that defined the RAG pattern: retrieval and generation as decoupled components. Included because it's the architecture, not the model math.
- Do: build a minimal retrieval pipeline yourself (plain embeddings + cosine similarity, no framework) before reaching for a vector database — understand what the abstraction is actually doing.
- Goal: know when retrieval beats a bigger context window (and vice versa), and understand failure modes — irrelevant/stale chunks, lost-in-the-middle, context poisoning from untrusted retrieved content.

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
- Read: [Raising the bar on SWE-bench Verified](https://www.anthropic.com/engineering/swe-bench-sonnet) — Anthropic; case study in minimal scaffolding + agent-computer interface (tool doc/spec) design for a coding agent
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

## Phase 5: Case Studies (Reference Architectures)

Real-world examples of complex agentic systems. Placed last deliberately — these are worked systems that combine tool reliability, evals, and production concerns, so they read very differently once those phases are behind you than they would as a Phase 1 primer.

- Explore: [Anthropic Financial Services](https://github.com/anthropics/financial-services) — a comprehensive blueprint for vertical agents (Investment Banking, Research, etc.) using modular skills and MCP connectors.
- Explore: [awesome-agentic-ai-zh](https://github.com/WenyuChiou/awesome-agentic-ai-zh) — curated (Chinese-language) list of agentic AI resources.
- Explore: [ai-agent-book](https://github.com/bojieli/ai-agent-book) — book-length treatment of AI agent design.
- Explore: [Agentic Design Patterns](https://github.com/evoiz/Agentic-Design-Patterns) — Antonio Gulli's hands-on guide covering foundational, advanced, and production agent patterns with code notebooks.
- Explore: [AI Agents for Beginners](https://github.com/microsoft/ai-agents-for-beginners) — Microsoft's official intro course/repo on building AI agents.
- Goal: study how to move from generic "chat" to specialized, tool-heavy workflows — now with a working vocabulary for evaluating *why* each architecture makes the reliability/eval/systems tradeoffs it does.

---

## Career Context

- Core LLM development is consolidating at a few labs — not the growth path for most engineers
- Application layer (agents, reliability, evals) is where most value is created and where talent is scarce
- Durable edge: understanding *why* systems fail, not just how to wire them up
- At Google: production-scale AI problems are better training ground than most places

---

## Current Progress

### General
- [x] Read Mitchell Hashimoto's [AI Adoption Journey](https://mitchellh.com/writing/my-ai-adoption-journey)

### Phase 1: Core Concepts
- [x] Understood agent/MCP/harness/skill conceptually
- [ ] Read "ReAct: Synergizing Reasoning and Acting in Language Models"
- [ ] Read "Plan-and-Solve Prompting" (Planning pattern)
- [ ] Analyze the [leaked Claude Code repo](https://github.com/codeaashu/claude-code)
- [ ] Analyze the [VideoCode repo](https://github.com/MarkTechStation/VideoCode) (Agent study)
- [x] Read "Building effective agents" (Anthropic) — very helpful
- [x] Read basic workflow patterns cookbook
- [ ] Study "Plan-and-Execute" implementations (LangGraph/LangChain)
- [ ] Read MCP spec
- [ ] Build a minimal MCP server
- [ ] Wire MCP server into Claude Code
- [ ] Read OpenAI's [Harness Engineering](https://openai.com/index/harness-engineering/) blog post
- [ ] Read Martin Fowler's [Harness Engineering Memo](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering-memo.html)
- [ ] Read LangChain's [Anatomy of an Agent Harness](https://www.langchain.com/blog/the-anatomy-of-an-agent-harness)
- [ ] Read Anthropic's [Effective Harnesses](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [ ] Read Anthropic's [Harness Design](https://www.anthropic.com/engineering/harness-design-long-running-apps)
- [ ] Read Anthropic's [Managed Agents](https://www.anthropic.com/engineering/managed-agents)
- [ ] Read [Claude Managed Agents overview](https://platform.claude.com/docs/en/managed-agents/overview) (docs)
- [ ] Read Anthropic's [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [ ] Read Anthropic's [Contextual Retrieval](https://www.anthropic.com/engineering/contextual-retrieval)
- [ ] Read the RAG paper (Lewis et al., 2020)
- [ ] Build a minimal retrieval pipeline (raw embeddings + cosine similarity)

### Phase 2: Tool Reliability
- [ ] Read "Raising the bar on SWE-bench Verified" (Anthropic)
- [ ] Build a tool with retry + idempotency

### Phase 3: Evaluation
- [ ] Set up promptfoo, write first eval suite

### Phase 4: Systems Thinking
- [ ] (none started yet)

### Phase 5: Case Studies
- [ ] Analyze [Anthropic Financial Services](https://github.com/anthropics/financial-services) architecture
- [ ] Study [awesome-agentic-ai-zh](https://github.com/WenyuChiou/awesome-agentic-ai-zh)
- [ ] Study [ai-agent-book](https://github.com/bojieli/ai-agent-book)
- [ ] Study [Agentic Design Patterns](https://github.com/evoiz/Agentic-Design-Patterns) (Antonio Gulli)
- [ ] Study [AI Agents for Beginners](https://github.com/microsoft/ai-agents-for-beginners) (Microsoft)
