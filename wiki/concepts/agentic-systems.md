# Agentic Systems

**What it is**: LLM systems that autonomously select and invoke tools across multiple steps to complete a task, rather than generating a single response. The LLM is the *controller*; tools are the *actuators*.

The core insight: most real tasks can't be answered with a single forward pass. They require planning, retrieval, execution, and correction in a loop. An agent wraps this loop around an LLM.

---

## ReAct Pattern

**ReAct** (Reason + Act) is the fundamental control pattern for agents:

1. **Thought**: LLM reasons about what to do next (visible to system; may be hidden from user)
2. **Action**: LLM selects a tool and provides arguments
3. **Observation**: Tool executes and returns result
4. **Repeat** until the task is complete or a limit is reached

The reasoning step is what separates agents from simple function-calling pipelines. The LLM sees its own prior reasoning, the tool outputs, and the conversation history — and decides whether to act again or return a final answer.

---

## LangGraph Orchestration

**LangGraph** is a framework (by LangChain) for building stateful, multi-step LLM agents as directed graphs. Each node is a processing step; edges represent transitions between steps (optionally conditional).

Why graph-structured orchestration?
- **State persistence**: each node passes structured state forward; nothing is lost between steps
- **Branching**: conditional edges let you route based on tool output (e.g. "if Cypher fails, try VectorRAG")
- **Memory module**: conversation history stored in state across turns
- **Token streaming**: output appears in real time, not after full generation

Both papers from the TU Delft group (Bunkova 2026, Alimin 2026) use LangGraph as their agent backbone.

---

## Tool Selection

The agent's core decision: given a query and available tools, which tool to invoke?

In the ChatP&ID 4-tool setup:
- ContextRAG → best overall; chosen for most queries
- VectorRAG → fast attribute lookup; chosen when precision matters more than traversal
- PathRAG → flow/path questions; chosen when routing/connectivity is the point
- CypherRAG → summary and structured queries; chosen when the question maps cleanly to a graph pattern

Tool selection quality degrades with smaller models — they may pick the wrong tool or loop without making progress. Mitigation: explicit tool descriptions, max-iteration limits, fallback logic.

---

## Memory in Agents

Two kinds of memory matter:
- **In-context memory**: conversation history passed as part of the prompt (ephemeral; lost after session)
- **External memory**: persisted state in a database or file (durable; survives sessions)

The ChatP&ID system uses an in-context memory module that accumulates turns within a session. For long conversations, this becomes expensive — a reason to consider summarization or compression.

---

## Key deployment constraints

From ChatP&ID failure mode analysis:
1. **Tool-call loops**: agents without hard limits loop indefinitely — always set `max_iterations`
2. **Output format drift**: LLMs generate verbose answers without explicit schema — always specify output format
3. **Caching**: static artifacts (graph context, schema) should be prompt-cached — re-embedding on every call is expensive
4. **Model size**: smaller models need simpler tool outputs; CypherRAG requires frontier models; VectorRAG/PathRAG degrade more gracefully

---

## Relevance to El Agente Gráfico

El Agente Gráfico will be one component in the El Agente architecture. The agentic pattern defines how it should work:
- Expose a set of tools (diagram parsing, KG query, layout rendering)
- Use LangGraph to orchestrate multi-step tasks
- Implement memory for session continuity
- Set hard limits and explicit output schemas from the start

---

## Papers
- [[sources/alimin2026-chatp-id]] — LangGraph agent with 4-tool GraphRAG; memory module; token streaming; failure modes documented
- [[sources/bunkova2026-kg-llm-synthesis]] — LangGraph agent for Text2Cypher over reaction KGs; same framework, different domain
