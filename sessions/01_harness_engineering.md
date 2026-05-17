# Session 1: Harness Engineering

Derived from: 
- [Harness Engineering: Leveraging Codex in an Agent-First World](https://openai.com/index/harness-engineering/) (OpenAI)
- [Harness Engineering Memo](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering-memo.html) (Martin Fowler)
- [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) (Anthropic)
- [Harness Design for Long-Running Apps](https://www.anthropic.com/engineering/harness-design-long-running-apps) (Anthropic)

## The Core Shift
Stop trying to "fix the prompt." Start fixing the **environment** (the harness). If the agent fails, it's usually because it lacks the necessary **legibility, observability, or tools**.

---

## 1. Agent Legibility (Repo-as-Context)
Agents reason best when the system's "intent" is codified in the repository, not just in human minds.

- **In-Repo Knowledge**: Move architecture maps, core beliefs, and design decisions into Markdown files.
- **Task**: Audit this repository. If a fresh agent started here, could it determine the project's goal and current progress just from `README.md` and `GEMINI.md`?
- **Action**: Ensure every architectural "rule" is caught by a linter or documented in a way an agent can find.

## 2. Agent Observability
Most agents are "blind" to the runtime. To debug, they need the same tools humans use.

- **Direct Access**: Give agents the ability to query logs (LogQL) or metrics (PromQL) directly.
- **UI Feedback**: For web agents, providing a "screenshot" skill or "DOM snapshot" is critical for closing the feedback loop.
- **Exercise**: If you were building a tool to fix bugs in a backend service, what specific "observability tools" would the agent need to see the stack trace autonomously?

## 3. Rigid Architecture (The Guardrails)
Humans find boilerplate annoying; agents find it helpful.

- **Strict Layering**: Enforce patterns (e.g., Types → Repo → Service) that limit the "state space" an agent has to explore.
- **Linter-Driven Taste**: Use custom lint rules to enforce architectural "taste" and prevent "AI slop" (redundant code, inconsistent naming).
- **The "Garbage Collector"**: Run background agents specifically to refactor debt and align code with new architectural patterns.

## 4. The "Ralph Wiggum Loop"
Agents should never submit work without self-critique.

- **Multi-Agent Review**: Have one agent implement and another (or the same one in a different pass) review against the project's standards.
- **Feedback Loops**: The agent should iterate until all automated checks (lint, test, build) pass before human intervention.

---

## Practical Experiment: "Legibility Audit"
1. **Scenario**: A new agent is tasked with adding a "Phase 5" to the roadmap.
2. **Current State**: Does the `README.md` provide enough context on the "Philosophy" to ensure Phase 5 aligns?
3. **Improvement**: Add a `ARCHITECTURE.md` or update `GEMINI.md` with explicit "rules of the road" for roadmap expansion.
