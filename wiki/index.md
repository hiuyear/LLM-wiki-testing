# Wiki Index

## Sources
| Page | Summary |
|---|---|
| [bunkova2026-kg-llm-synthesis](sources/bunkova2026-kg-llm-synthesis.md) | Text2Cypher pipeline over a reaction KG for synthesis retrieval; one-shot prompting beats zero-shot; CoVe validator is the bottleneck |
| [alimin2026-chatp-id](sources/alimin2026-chatp-id.md) | GraphRAG over P&IDs via 4-tool agent (ContextRAG/VectorRAG/PathRAG/CypherRAG); conceptual KG = 7K tokens, 91% accuracy at $0.004/task |

## Concepts
| Page | Summary |
|---|---|
| [text2cypher](concepts/text2cypher.md) | Translating NL to Cypher graph queries; directionality and semantic vs. syntactic validity are key challenges; CypherRAG is a production instantiation |
| [kg-grounding-vs-rag](concepts/kg-grounding-vs-rag.md) | KG-grounding vs. RAG vs. GraphRAG: tradeoffs and when to use each; chemistry/engineering agents likely need all three |
| [chain-of-verification](concepts/chain-of-verification.md) | CoVe self-correction loop: generate → validate → correct; validator quality is the bottleneck, not the corrector |
| [graphrag](concepts/graphrag.md) | Graph-based RAG: 4-tool architecture (ContextRAG/VectorRAG/PathRAG/CypherRAG); semantic enrichment layer; graph abstraction levels; failure modes |
| [agentic-systems](concepts/agentic-systems.md) | ReAct pattern, LangGraph orchestration, tool selection, memory modules; deployment constraints from empirical evaluation |

## Entities
| Page | Summary |
|---|---|
| [uspto-dataset](entities/uspto-dataset.md) | Large-scale patent reaction dataset in SMILES format; standard benchmark for reaction prediction and synthesis planning |
| [dexpi-standard](entities/dexpi-standard.md) | Open standard for P&ID representation as semantic XML; pyDEXPI parses to KG; raw Proteus = 150K tokens → conceptual KG = 7K tokens |
