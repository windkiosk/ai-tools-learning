# Session 2: Loop Engineering vs. Graph Engineering

Derived from:
- [Loop Engineering vs. Graph Engineering: The Architecture Shift Quietly Reshaping AI Agents](https://medium.com/@neuraldev/loop-engineering-vs-graph-engineering-the-architecture-shift-quietly-reshaping-ai-agents-c83488435d23) (Uday Sharma)
- [Agent Harness Engineering vs. Loop Engineering vs. Graph Engineering](https://medium.com/@bijit211987/agent-harness-engineering-vs-loop-engineering-vs-graph-engineering-44a967d6b975) (Bijit Ghosh)
- [3 Years of Graph Engineering with LangGraph](https://www.langchain.com/blog/3-years-of-graph-engineering-with-langgraph) (LangChain)
- [Architectural Patterns for Loop Engineering](https://dr-arsanjani.medium.com/architectural-patterns-for-loop-engineering-843eadc289df) (Ali Arsanjani)
- [From Agent Loops to Structured Graphs: A Scheduler-Theoretic Framework for LLM Agent Execution](https://arxiv.org/pdf/2604.11378)

## The Core Shift

Harness engineering (Session 1) builds the *environment* an agent runs in. This session is one level up: given a harness, how do you shape the agent's *control flow* — the thing that decides what happens next? Two answers dominate in practice, and they trade off differently:

- **Loop engineering**: a single agent runs discover → plan → execute → verify on repeat, all state living in one ever-growing transcript/context window.
- **Graph engineering**: the workflow's shape is made explicit — nodes, branches, joins, controlled cycles — with typed state per node instead of an implicit transcript.

Neither is "more advanced" than the other. They're different answers to "where does process state live, and how visible is control flow to a human debugging it?"

---

## 1. The Loop, and Why It's the Default

A single ReAct-style loop (perceive → think → act → repeat) is cheap to build, cheap to debug, and — per the "10 loop engineering patterns" survey — handles more real-world tasks than most builders expect before it needs to be replaced.

- **State model**: everything the agent has tried, failed at, or learned lives inside the context window as a transcript. There's no separate "memory of the process" — just the conversation itself.
- **Failure mode**: as the transcript grows, the loop has no way to discard irrelevant history without also losing the parts that mattered. A failed sub-step forces the *whole* loop to re-reason from scratch, because there's no addressable checkpoint to rewind to.
- **When to reach for it**: single-role tasks, short-to-medium horizons, anything where "just re-run the loop with a note about what failed" is an acceptable retry strategy.

Common hardening patterns layered on top of a bare loop (from Arsanjani's catalog): retry with backoff, circuit breakers, bounded execution (max iterations), and "never grade your own homework" (a separate verifier, not the same pass that produced the output).

## 2. The Graph, and What It Buys You

LangGraph's framing: model execution as a stateful, cyclic directed graph rather than a single loop. Concretely:

- **Nodes as roles**: different nodes can be different "expert" agents, each scoped to one job — closer to a real team's division of labor than one generalist looping forever.
- **Typed state, not transcript**: state updates are explicit and per-node, so a node doesn't have to re-derive what happened by re-reading a growing conversation — it reads a defined state object.
- **Localized retry**: when one step fails, only that step's edge sends work back to be redone. The rest of the graph's state stays untouched, instead of the entire process looping blindly from the top.
- **Debuggability**: because the workflow is a map (nodes + edges), it's possible to point at exactly where something went wrong, insert a guard node or human-approval step at a specific edge, and reason about the system without replaying an entire transcript.

The cost: more upfront design (you have to know your workflow's shape before you build it), and more moving parts to maintain than a single loop.

## 3. Choosing Between Them

The scheduler-theoretic framing (arXiv 2604.11378) is useful here: a loop is a degenerate graph with one node and one self-edge. So the real question isn't "loop or graph" as a binary — it's **at what point does an implicit single-node loop stop being legible enough**, and you need to factor it into an explicit multi-node graph?

Signals it's time to move from loop → graph:
- Multiple distinct roles/expertise areas are being crammed into one system prompt.
- You need to insert an approval/guard step at a *specific* point, not "somewhere in the transcript."
- Retries are wasting work — a failure late in the process forces a full restart instead of resuming from a checkpoint.
- Debugging requires a human to read the entire transcript to find where things went wrong, rather than pointing at a node.

---

## Practical Exercise

1. **Pick an agent you've built or sketched** (or use Claude Code's own tool-call loop as the subject).
2. **Trace one failure**: identify a case where a sub-step failed partway through a run. Under the current (loop) model, what got re-run or re-reasoned that didn't need to be?
3. **Redraw it as a graph**: sketch the nodes and edges that failure would have touched if the workflow were an explicit graph instead. Would the retry have been cheaper? Would a human debugging it have found the failure point faster?
4. **Judgment call**: would you actually migrate this agent to a graph, or is the loop still the right tool here? Write one sentence justifying the call — this is the kind of decision worth capturing per [[Harness legibility]] (GEMINI.md rule 4), not just deciding in your head.
