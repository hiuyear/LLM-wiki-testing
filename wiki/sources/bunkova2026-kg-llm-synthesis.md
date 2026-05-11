# Grounding LLMs in Reaction Knowledge Graphs for Synthesis Retrieval

**Authors**: Olga Bunkova, Lorenzo Di Fruscia, Sophia Rupprecht, Artur M. Schweidtmann, Marcel J.T. Reinders, Jana M. Weber  
**Institution**: Delft University of Technology  
**Venue**: ML4Molecules 2025 (ELLIS UnConference), Copenhagen — December 2, 2025  
**arXiv**: 2601.16038v1 (submitted January 22, 2026)  
**Code**: https://github.com/Intelligent-molecular-systems/KG-LLM-Synthesis-Retrieval  

---

## Problem

LLMs hallucinate and have stale knowledge when used for chemical synthesis planning. Standard RAG partially helps but treats documents in isolation — no connected view of molecules and reactions. This paper asks: can LLMs query a structured **reaction knowledge graph** directly via natural language, getting grounded, verifiable answers?

---

## Approach

Cast reaction path retrieval as a **Text2Cypher** problem: translate a natural language chemistry question into a Cypher query that runs against a Neo4j knowledge graph.

### Knowledge Graph Construction
- Source data: **USPTO** reaction dataset (~50k reactions sampled), preprocessed with ORDerly
- Molecules represented as canonicalized SMILES strings
- **Bipartite graph** structure: (:Molecule) and (:Reaction) nodes alternate, connected by typed edges:
  - `(:Molecule)-[:REACTS_IN]->(:Reaction)`
  - `(:Reaction)-[:PRODUCES]->(:Molecule)`
  - `(:Reaction)-[:USES_AGENT]->(:Molecule)`
  - `(:Reaction)-[:USES_SOLVENT]->(:Molecule)`
- Stored in Neo4j

### Tasks
| Type | Description | Queries |
|---|---|---|
| Single-step | One-hop retrieval: reactants, products, agents/solvents of a reaction | 1200 (6 types × 200) |
| Multi-step | Path retrieval: reaction chains of length 2, 3, or 4 | 1200 (4 types × 300) |

### Prompting Strategies Compared
- **Zero-shot (ZS)**: no example
- **One-shot static (1S-S)**: same fixed exemplar for all queries
- **One-shot random (1S-D-R)**: random exemplar per query
- **One-shot semantic (1S-D-S)**: exemplar selected by cosine similarity in embedding space (all-mpnet-base-v2); SMILES masked so similarity reflects task intent, not chemistry

Also tested: **Chain-of-Verification (CoVe)** self-correction loop — LLM generates query → validator checks against fixed checklist → corrector applies minimal edits → repeat up to 3 times.

**LLM used**: GPT-4.1-mini-2025-04-14 (T=0)  
**Orchestration**: LangChain + LangGraph

---

## Key Findings

### 1. Text-to-text similarity is a bad proxy for retrieval accuracy
BLEU/METEOR/ROUGE-L between generated and reference Cypher queries do not correlate well with whether the query actually retrieves the right answer. Two reasons:
- Multiple valid Cypher formulations can be semantically equivalent
- Small syntactic changes (e.g. losing edge directionality) keep BLEU high while breaking retrieval entirely

**Implication**: always evaluate on *execution results*, not query surface form.

### 2. Zero → one-shot is the biggest gain, especially for multi-step
In multi-step tasks, zero-shot LLMs commonly make two errors:
- **Endpoint anchoring**: treating the target molecule as the start of the path (wrong direction)
- **Traversal direction violation**: misunderstanding the Mol→Reac→Mol bipartite alternation

One example in the prompt largely eliminates both. After that, choosing a *better* exemplar (semantic vs random) gives only marginal additional gain.

### 3. CoVe self-correction has limited benefit; validator is the bottleneck
CoVe helps mainly in zero-shot settings by fixing *missing reaction components* (reactants/products not included). In one-shot settings, improvement is negligible. The bottleneck is the **validator**, not the corrector: a generic checklist misses 86–95% of task-specific errors (e.g. duplicate or incomplete molecules). Task-specific/schema-aware validators are recommended.

---

## Relevance to El Agente

This paper is directly in scope. El Agente needs to query and reason over structured chemical knowledge (reactions, molecules, synthesis routes). The KG-grounding approach here — using structured graph queries rather than unstructured RAG — is a strong architectural direction for chemistry agents. Key takeaways for design:
- Structure the knowledge base as a graph, not a flat document store
- One-shot prompting with intent-aligned exemplars is cheap and effective
- Evaluation must be execution-grounded, not surface-level

---

## Cross-references
- Concept: [[concepts/text2cypher]]
- Concept: [[concepts/kg-grounding-vs-rag]]
- Concept: [[concepts/chain-of-verification]]
- Entity: [[entities/uspto-dataset]]
