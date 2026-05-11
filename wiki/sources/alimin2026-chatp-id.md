# GraphRAG for Engineering Diagrams: ChatP&ID Enables LLM Interaction with P&IDs

**Authors**: Achmad Anggawirya Alimin, Artur M. Schweidtmann  
**Institution**: Process Intelligence Research Group, Dept. of Chemical Engineering, Delft University of Technology  
**arXiv**: 2603.22528v1 (submitted March 25, 2026)  
**Note**: Schweidtmann is also co-author on [[sources/bunkova2026-kg-llm-synthesis]] — same group, same stack.

---

## Problem

P&IDs (Piping and Instrumentation Diagrams) are the central blueprints of chemical process facilities — describing equipment, piping, control logic, and safety elements. But they're stored in static formats (PDFs, images, or CAD files). Feeding them to LLMs directly is broken in three ways:
- **Raw images**: 18% lower accuracy than KG-based methods; LLM vision encoders downsample and misread text
- **Raw DEXPI XML (Proteus files)**: up to 150K tokens per page; semantically noisy (URIs, metadata); 85% more expensive than conceptual-level KG
- **Both**: hallucination risk because LLMs fill in gaps from parametric memory

The question: can we represent a P&ID as a structured knowledge graph and use GraphRAG to enable accurate, cheap, grounded natural-language interaction?

---

## Approach: ChatP&ID

Three-layer architecture:

### 1. KG Generation (pyDEXPI → Neo4j)
- DEXPI smart P&IDs are parsed via **pyDEXPI** (open-source Python library from the same group)
- Each P&ID component → node; relationships (compositional, reference) → edges
- Format: **Labeled Property Graph (LPG)**, stored in Neo4j
- Three abstraction levels, from most to least detail:
  - **Complete**: one-to-one mapping from pyDEXPI (full hierarchy, noisy)
  - **Process**: collapses piping segments into components
  - **Conceptual**: further collapses instrumentation and equipment — 7K tokens vs 150K for Proteus file

### 2. Semantic Enrichment + Embedding
- For each node, GPT-4o generates two text descriptions: **global semantic** (role in full flowsheet) and **local semantic** (role relative to immediate neighbors)
- Both encoded as 1024-dim vectors using **Voyage-3.5-lite** embedding model
- Stored back in Neo4j; used by VectorRAG and PathRAG at query time

### 3. Four GraphRAG Tools (LLM-invokable)
| Tool | Mechanism | Best for | Avg accuracy | Cost/task |
|---|---|---|---|---|
| **ContextRAG** | Serialize KG to filtered GraphML → pass as LLM context | All tasks | 0.91 | $0.004 |
| **VectorRAG** | Cosine similarity over global node embeddings → top-k nodes | Precise attribute lookup | 0.82 | $0.002 |
| **PathRAG** | VectorRAG for start node → iterative graph traversal using local embeddings | Path/flow tracing | 0.83 | $0.002 |
| **CypherRAG** | LLM generates Cypher → execute on Neo4j → answer | Summarization, structured queries | 0.86 | $0.002 |

**ContextRAG two modes**: Graph mode (full attributes) vs Topology mode (only connectivity — lightweight).

### Agentic Framework
- Built with **LangGraph** (same stack as paper 1)
- Agent receives query → selects tool → executes → retrieves context → generates answer → iterates if needed
- Memory module persists conversation history across turns
- Token streaming for real-time UX

---

## Key Results

**Best overall configuration**: GPT-5-mini + ContextRAG = **91% accuracy at $0.004/task**

**Graph vs. raw formats** (GPT-5, conceptual graph vs. Proteus):
- +5% accuracy, 85% cost reduction (7K vs 150K tokens)
- Larger models tolerate raw formats better; smaller models need clean abstraction to function

**Tool performance** (all with GPT-5-mini):
- ContextRAG dominates on all task types
- CypherRAG strong on summarization (1.00) and single queries (0.88) but model-dependent — only frontier models reliably generate valid Cypher
- PathRAG best for path exploration; slower (54.6s avg) due to iterative traversal
- VectorRAG fastest for attribute lookup

**Offline models (Ollama)**: struggle with ContextRAG (complex GraphML), but VectorRAG/PathRAG improve accuracy 20–40% by offloading decision-making to deterministic algorithms

**Semantic similarity vs. LLM-as-judge**: Semantic similarity fails for factual queries — a verbose correct answer gets penalized for being "far" from a short reference answer. LLM-as-judge with rubric (correctness, completeness, coherence, relatedness) is more reliable. Same finding as paper 1.

---

## Common Failure Modes (directly useful for El Agente Gráfico)

1. **Model too small**: can't parse complex GraphML or XML structures
2. **Semantic noise**: URIs, raw identifiers in context confuse smaller models
3. **Context too long**: filling the full context window doesn't always help, often hurts
4. **Infinite tool loops**: agents may loop unproductively without tool-call limiters
5. **Underspecified output format**: LLMs "yap" without explicit output schema
6. **Inefficient caching**: repeatedly passing large static artifacts per inference — use prompt caching

---

## Relevance to El Agente Gráfico

This paper is a near-direct blueprint for El Agente Gráfico. Key takeaways:
- The 4-tool GraphRAG architecture (Context/Vector/Path/Cypher) is a production-ready pattern for querying structured process data
- Conceptual-level KG abstraction is the sweet spot: high accuracy, low cost, small model-friendly
- LangGraph agent with tool selection + iteration is the right framework choice
- The failure mode list is a practical checklist for deployment

---

## Cross-references
- Concept: [[concepts/graphrag]]
- Concept: [[concepts/agentic-systems]]
- Concept: [[concepts/kg-grounding-vs-rag]] (updates this page)
- Concept: [[concepts/text2cypher]] (CypherRAG is an implementation)
- Entity: [[entities/dexpi-standard]]
- Related source: [[sources/bunkova2026-kg-llm-synthesis]] (same group, same stack, different domain)
