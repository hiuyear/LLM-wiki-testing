# Knowledge Graph Grounding vs. RAG

Two architectures for connecting LLMs to external knowledge. Understanding the tradeoff is important for designing chemistry AI agents.

---

## RAG (Retrieval-Augmented Generation)

The standard pattern. At query time: embed the question → find similar document chunks → stuff them into context → LLM generates answer.

**Strengths**: works on unstructured text, easy to set up, flexible  
**Weaknesses**:
- Retrieval is over *chunks*, not *relationships* — multi-hop reasoning requires finding and stitching together fragments
- No persistent structure — the LLM rediscovers connections every time
- Hallucination risk: LLM may generate facts not in the retrieved chunks
- Doesn't preserve long-range links (e.g. a 4-step synthesis route)

---

## KG Grounding (Knowledge Graph Grounding)

Instead of retrieving text chunks, the LLM generates a *structured query* (Cypher, SPARQL, SQL) that runs against a graph database. The graph encodes entities and their typed relationships explicitly.

**Strengths**:
- Multi-hop traversal is native: "find precursors 3 reactions back" is a path query
- Answers are execution-grounded — retrieved from verified data, not generated
- Preserves global structure (the entire reaction network is connected)
- No hallucination on retrieved facts (though query generation can still be wrong)

**Weaknesses**:
- Requires structured data and schema design upfront
- LLM must generate valid queries — error-prone, especially zero-shot
- Harder to scale to unstructured sources

---

## GraphRAG: the middle path

**GraphRAG** combines both approaches: embed graph *nodes* (not text chunks), retrieve them by semantic similarity or traversal, then pass the retrieved subgraph as context to an LLM. It inherits:
- From RAG: semantic similarity search, natural-language queries
- From KG-grounding: structured relational context, multi-hop traversal, grounded facts

The 4-tool breakdown (from ChatP&ID):
- **ContextRAG**: serialize subgraph → LLM context (highest accuracy; best when the graph fits in context)
- **VectorRAG**: cosine similarity over node embeddings → top-k nodes (fast attribute lookup)
- **PathRAG**: VectorRAG for entry + iterative traversal (best for path/flow questions)
- **CypherRAG**: NL → Cypher → execute → answer (grounded; requires frontier model to generate valid queries)

See [[concepts/graphrag]] for full detail.

---

## For chemistry and engineering specifically

Chemical knowledge is highly relational: molecules, reactions, reagents, solvents, products all interconnect. RAG over paper text misses the structure. A KG encodes it explicitly. Bunkova et al. (2026) show that KG-grounded retrieval reliably returns correct multi-step synthesis routes — something flat RAG struggles with.

Engineering diagrams (P&IDs) are similarly relational: equipment, piping, and control logic form a graph. ChatP&ID (Alimin & Schweidtmann 2026) shows GraphRAG over DEXPI-parsed P&IDs achieves 91% accuracy at $0.004/task — vs. 18% lower accuracy and 85% higher cost for raw image/XML approaches.

**The hybrid view**: RAG, KG-grounding, and GraphRAG are complementary. RAG handles unstructured literature. KG-grounding handles precise structured queries. GraphRAG handles the middle ground where structure exists but queries are open-ended. A production chemistry agent likely needs all three.

---

## Papers
- [[sources/bunkova2026-kg-llm-synthesis]] — empirical comparison of prompting strategies for KG-grounded synthesis retrieval
- [[sources/alimin2026-chatp-id]] — GraphRAG over P&IDs; 4-tool architecture; conceptual KG abstraction as cost-accuracy sweet spot
