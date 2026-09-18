# Interview Prep: Anthropic Backend / Full-Stack SWE

Companion to `README.md`. The README is the AI-engineering *learning* roadmap; this file is the *interview* plan. It lives separately so the roadmap stays a curriculum, not a job-hunt log.

## Context (why this file exists)

- **Target**: backend-focused (or full-stack) software engineer roles at Anthropic. Preference stated 2026-09-18: backend first, full-stack acceptable.
- **Known baseline**: needs to refresh LeetCode-style coding and system design. The AI-topic material is already tracked in `README.md`.
- **Example JDs reviewed** (examples of the role family, not the specific target; Anthropic postings change, so re-check the live listing):
  - [Senior Software Engineer, Full-stack](https://job-boards.greenhouse.io/anthropic/jobs/5174743008): Product Eng org. 6+ years full-stack, React/TypeScript, 0→1 ownership, cross-team influence. Teams: Developer Experience, Beneficial Deployments, Vertical AI Products, Enterprise AI (MCP, skills, connectors, retrieval), Public Sector, Enterprise Foundations (identity, permissions, compliance, admin analytics), Growth. Team placement happens *after* the interview.
  - [Staff Software Engineer, Claude Code](https://job-boards.greenhouse.io/anthropic/jobs/5383610008): agentic dev tools, tool use / orchestration / prompt engineering, collaboration with researchers on tools and evals, strict safety/security/compliance experience, React, CLIs, containers, Bun/Deno.
- **Decision record**: an early draft of this plan assumed an *infra* role (LLM inference serving, accelerators, Kubernetes internals). Both example JDs are product/platform engineering instead, so that track is deprioritized. See "Only if the role turns out to be infra" at the bottom.
- **Unverified**: the claim that coding rounds skew practical (build something with tests) rather than pure puzzles is candidate hearsay, not confirmed. Ask the recruiter for the loop format and treat this file's coding section as a hedge across both styles.

## Suggested time split

| Area | Share |
|---|---|
| Coding (LeetCode + practical) | ~40% |
| Backend system design | ~30% |
| AI/agent knowledge (from `README.md`) | ~15% |
| Behavioral + Anthropic-specific | ~15% |

Shift toward system design if coding already feels sharp.

## 1. Coding

Pick one primary language and be fluent in it. Python is the safest default for Anthropic backend work; check the specific JD's stack (Go, Rust, TypeScript also appear).

- [ ] Pattern refresh (mediums): hash maps, two pointers / sliding window, heaps, graphs (BFS/DFS/topological sort), intervals, binary search, trees, DP basics
- [ ] Concurrency-flavored problems: rate limiter (token bucket / sliding window), LRU/LFU cache, task scheduler, thread-safe queue
- [ ] Practical drill: build a small service or library with tests in ~1 hour
- [ ] Practical drill: parse and aggregate a large log stream
- [ ] Practical drill: retry with exponential backoff + jitter, with a clean failure mode

## 2. Backend system design (LLM-platform flavored)

LLM workloads differ from ordinary web services: requests are long-lived, expensive, variable in cost, and often streamed. Practice these designs end to end (requirements, API, data model, scaling, failure modes):

- [ ] Token-based rate limiting and quotas for multi-tenant traffic (per-request limits fail when request cost varies widely)
- [ ] Streaming API: SSE, cancellation, mid-stream failure, resumable connections
- [ ] Async / batch job system: queues, priorities, retries, dead-letter handling
- [ ] Usage metering and billing: exactly-once accounting from at-least-once events
- [ ] Enterprise auth: RBAC, SSO/SCIM, audit logs, tenant isolation
- [ ] Long-running agent sessions: durable state, resume after crash, sandboxed code execution
- [ ] Caching layers, including prompt caching from the operator's point of view
- [ ] Generic staples still worth one pass: URL shortener, notification system, feed, distributed cache

## 3. Backend fundamentals to refresh

- [ ] Postgres: transactions, isolation levels, indexing, online schema migrations
- [ ] Redis and Kafka-style queues: ordering, partitioning, consumer groups, replay
- [ ] Distributed systems: idempotency, delivery guarantees, consistency tradeoffs, backpressure, load shedding, circuit breakers
- [ ] Reliability: SLOs/error budgets, graceful degradation, incident response, postmortem writing
- [ ] User-level infra: containers, Kubernetes basics, observability (logs, metrics, traces)

## 4. AI/agent knowledge, mapped from `README.md`

| README section | Priority for backend interviews | Why |
|---|---|---|
| Phase 2: Tool Reliability | **High, start now** | Idempotency keys, retries, circuit breakers, error contracts are backend fundamentals in an agent context. Best interview material; actually build the retry + idempotency tool. |
| Phase 4: Systems Thinking | **High, do right after Phase 2** | Observability, cost (prompt caching, batching, routing), security. Currently the least-started phase. |
| Phase 1: Core Concepts | Medium | Know MCP, harness, context engineering well enough to *discuss*; no need to read every article. |
| Phase 3: Evals | Low-medium | Matters for researcher-collaboration roles (e.g. Claude Code), less for pure backend. |
| Phase 5: Case Studies | Low | Skip unless time is left over. |

## 5. Safety and security

Anthropic's JDs put security and safety at the center; expect it in design rounds too.

- [ ] Prompt injection and untrusted tool/retrieval content
- [ ] Tool permission scoping and sandboxing untrusted code
- [ ] Tenant isolation, secrets handling, audit trails
- [ ] Compliance-heavy environments (the Public Sector JD mentions FedRAMP and classified networks)

## 6. Behavioral and Anthropic-specific

- [ ] 5-6 stories: a production incident you drove, a scaling/reliability win, cross-team influence without authority, an ambiguous project, a mistake you owned
- [ ] A genuine answer to "why Anthropic" that reflects the safety mission
- [ ] Read Anthropic's engineering blog and the company's published safety framing
- [ ] Use the API as an operator would: prompt caching, Batch API, rate limits
- [ ] Read Anthropic's guidance on candidates' AI usage (linked from the JDs) *before* deciding how to use AI in prep or in the loop

## Open questions

- Which language for coding rounds?
- Which team / role family is the actual target (platform, enterprise foundations, developer tools)?
- Loop format from the recruiter (number of rounds, coding style, system design style)?

## Only if the role turns out to be infra

Lower priority for the roles above; keep as a conceptual checklist only.

- LLM serving: prefill vs. decode, KV cache, continuous batching, paged attention (vLLM paper), speculative decoding, quantization, TTFT vs. inter-token latency
- Accelerators: GPU vs. TPU vs. Trainium, HBM bandwidth, interconnect, collective comms
- Training infra: data/tensor/pipeline parallelism, checkpointing, fault tolerance, stragglers
- Kubernetes internals, cgroups/namespaces, sandboxing (gVisor, Firecracker)

Note this conflicts with the README's stance that model-internals theory is out of scope; the infra track needs the *systems* view of inference, not the math.
