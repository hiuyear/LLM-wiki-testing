# Text2Cypher

**What it is**: Translating natural language questions into Cypher queries that execute against a property graph database (typically Neo4j).

Cypher is the declarative query language for property graphs — analogous to SQL for relational databases. Text2Cypher is a specialization of the broader Text2SQL problem, but with graph-specific challenges: directionality of edges, multi-hop path patterns, bipartite alternation constraints.

---

## Why it matters for chemistry AI

Chemical knowledge is fundamentally relational — molecules react with other molecules to produce products via reactions that involve agents and solvents. A graph database encodes this naturally. If an LLM can generate correct Cypher queries from natural language, you get:
- **Grounded answers**: retrieved from verified data, not generated from parametric memory
- **Multi-hop reasoning**: graph traversal natively handles "what are the precursors 3 steps back?"
- **No hallucination on facts**: the KG is the source of truth

---

## Key challenges

**Directionality**: Cypher edges are directed. `(a)-[:REACTS_IN]->(b)` and `(a)<-[:REACTS_IN]-(b)` are different queries. LLMs frequently get this wrong in zero-shot settings.

**Structural validity vs. semantic correctness**: A query can be syntactically valid (executes without error) but semantically wrong (retrieves wrong results). Standard text-overlap metrics (BLEU, METEOR, ROUGE-L) don't catch this — you must evaluate on execution results.

**Exemplar alignment**: One-shot prompting helps enormously, but only if the exemplar shares the same Cypher *pattern type* as the query. An exemplar for forward synthesis doesn't reliably transfer to retrosynthesis.

---

## Prompting strategies (from Bunkova et al. 2026)

| Strategy | Description | Performance |
|---|---|---|
| Zero-shot | No example provided | Highest error rate, especially on multi-step |
| One-shot static | Same exemplar for all queries | Large gain over zero-shot |
| One-shot random | Random exemplar per query | Similar to static |
| One-shot semantic | Exemplar selected by cosine similarity in embedding space | Best overall; SMILES masked to focus on task intent |

---

## CypherRAG: Text2Cypher inside an agent tool

CypherRAG (from ChatP&ID) is Text2Cypher deployed as one tool in a 4-tool GraphRAG agent. Key differences from the standalone Text2Cypher pipeline in Bunkova et al.:

- The LLM *selects* whether to use CypherRAG vs. other tools (ContextRAG, VectorRAG, PathRAG)
- Operates over a P&ID knowledge graph (engineering diagrams) instead of a reaction KG
- No CoVe correction loop — query correctness depends entirely on the generating model

**Performance of CypherRAG** (GPT-5-mini):
- Summarization tasks: 1.00 accuracy (Cypher is well-suited to aggregation)
- Single-attribute queries: 0.88
- Fails silently with smaller models that generate syntactically invalid Cypher

The CypherRAG finding confirms what Bunkova et al. found: Text2Cypher is powerful but model-dependent. Frontier models (GPT-4o-class and above) generate reliable Cypher; smaller models need either CoVe correction (Bunkova approach) or should be routed to VectorRAG/ContextRAG instead.

---

## Papers
- [[sources/bunkova2026-kg-llm-synthesis]] — first chemistry-specific execution-grounded evaluation of Text2Cypher on reaction KGs; CoVe correction loop
- [[sources/alimin2026-chatp-id]] — CypherRAG as one tool in a 4-tool GraphRAG agent; P&ID domain; model-dependence confirmed
