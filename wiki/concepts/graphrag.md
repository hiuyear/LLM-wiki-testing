# GraphRAG

**What it is**: A family of retrieval architectures that combine a knowledge graph with LLM generation — the graph provides structured retrieval and traversal; the LLM provides natural-language understanding and answer synthesis.

GraphRAG is not a single technique. It's a pattern: instead of embedding chunks of text, you embed or query *graph nodes and edges* to retrieve grounded context. The LLM then generates answers from that structured context rather than from flat document chunks.

---

## The Four-Tool Architecture (from ChatP&ID, Alimin & Schweidtmann 2026)

The most complete production-ready instantiation. An agent is given four tools and selects among them per query:

| Tool | How it works | Strengths | Weaknesses |
|---|---|---|---|
| **ContextRAG** | Serialize the (filtered) KG to GraphML → pass as LLM context | Highest accuracy; handles all task types | Expensive on large graphs; requires good filtering |
| **VectorRAG** | Cosine similarity over global node embeddings → return top-k nodes | Fast for attribute lookup; works with small models | Misses relational/path structure |
| **PathRAG** | VectorRAG for entry node → iterative graph traversal using local embeddings | Best for flow/path tracing | Slowest (54.6s avg); iterative traversal cost |
| **CypherRAG** | LLM generates Cypher → execute on Neo4j → answer | Strong on structured/summary queries; grounded | Model-dependent; only frontier models generate valid Cypher reliably |

**ContextRAG two modes**:
- **Graph mode**: full node attributes serialized into GraphML
- **Topology mode**: connectivity only (lighter; useful when attribute details aren't needed)

**Best configuration observed**: GPT-5-mini + ContextRAG = 91% accuracy at $0.004/task.

---

## Semantic Enrichment Layer

ChatP&ID augments nodes with LLM-generated descriptions before embedding:
- **Global semantic**: what this node does in the context of the entire flowsheet
- **Local semantic**: what this node does relative to its immediate neighbors

These are 1024-dim vectors (Voyage-3.5-lite). Used by VectorRAG and PathRAG at query time. Without this enrichment, similarity search over raw node identifiers would be nearly meaningless.

---

## Graph Abstraction Levels

A key design decision: how much to collapse the raw data into the graph.

| Level | Description | Tokens (per P&ID page) |
|---|---|---|
| Complete | One-to-one from source (full hierarchy, noisy URIs) | ~150K |
| Process | Collapses piping segments into equipment components | ~50K |
| Conceptual | Further collapses instrumentation; clean labels only | ~7K |

The conceptual level is the practical default: 7K tokens vs 150K for raw XML. Smaller models that fail on Complete succeed on Conceptual. The tradeoff is loss of fine-grained attributes — acceptable for most natural-language queries.

---

## Why GraphRAG over plain RAG

| Dimension | Plain RAG | GraphRAG |
|---|---|---|
| Multi-hop reasoning | Requires stitching retrieved chunks | Native — graph traversal |
| Relational queries | Poor (chunks don't encode relations) | Strong |
| Factual grounding | Hallucination risk | Execution-grounded (VectorRAG, PathRAG, CypherRAG) |
| Unstructured text | Handles naturally | Requires structured graph first |
| Cost | Cheap | Depends on graph size and tool choice |

---

## Failure modes

From ChatP&ID's empirical results:
1. **Model too small** — can't parse GraphML or XML even at conceptual level
2. **Semantic noise** — URIs and raw identifiers confuse smaller models; use clean labels
3. **Context length** — filling the full context window often hurts more than it helps
4. **Tool-call loops** — agents without hard tool-call limiters loop unproductively
5. **Underspecified output** — LLMs produce verbose wrong answers without explicit output schema
6. **Caching inefficiency** — large static graph artifacts should be prompt-cached across calls

---

## Papers
- [[sources/alimin2026-chatp-id]] — the 4-tool GraphRAG instantiation; production evaluation on P&IDs
