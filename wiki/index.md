# Wiki Index

## Sources
| Page | Summary |
|---|---|
| [goldstein2025-pydexpi](sources/goldstein2025-pydexpi.md) | pyDEXPI: first open-source DEXPI implementation; 473 Pydantic data classes; import/export/toolkit; infrastructure layer for ChatP&ID |
| [bunkova2026-kg-llm-synthesis](sources/bunkova2026-kg-llm-synthesis.md) | Text2Cypher pipeline over a reaction KG for synthesis retrieval; one-shot prompting beats zero-shot; CoVe validator is the bottleneck |
| [alimin2026-chatp-id](sources/alimin2026-chatp-id.md) | GraphRAG over P&IDs via 4-tool agent (ContextRAG/VectorRAG/PathRAG/CypherRAG); conceptual KG = 7K tokens, 91% accuracy at $0.004/task |
| [heyer2025-rl-mechanistic-models](sources/heyer2025-rl-mechanistic-models.md) | RL (Q-learning) agent generates interpretable mechanistic reactor models from concentration data; 1.5× speedup over exhaustive search; NRMSE = 2.4% on Taylor-Couette reactor |
| [laub2026-rl-mechanistic-models-ii](sources/laub2026-rl-mechanistic-models-ii.md) | Part II: hierarchical RL for compartmentalization (graph grammar + colored digraphs) + ontology for knowledge management + Q-table transfer learning; auto-matches literature TCR models at ~5% RMSD |

## Concepts
| Page | Summary |
|---|---|
| [text2cypher](concepts/text2cypher.md) | Translating NL to Cypher graph queries; directionality and semantic vs. syntactic validity are key challenges; CypherRAG is a production instantiation |
| [kg-grounding-vs-rag](concepts/kg-grounding-vs-rag.md) | KG-grounding vs. RAG vs. GraphRAG: tradeoffs and when to use each; chemistry/engineering agents likely need all three |
| [chain-of-verification](concepts/chain-of-verification.md) | CoVe self-correction loop: generate → validate → correct; validator quality is the bottleneck, not the corrector |
| [graphrag](concepts/graphrag.md) | Graph-based RAG: 4-tool architecture (ContextRAG/VectorRAG/PathRAG/CypherRAG); semantic enrichment layer; graph abstraction levels; failure modes |
| [agentic-systems](concepts/agentic-systems.md) | ReAct pattern, LangGraph orchestration, tool selection, memory modules; deployment constraints from empirical evaluation |
| [digital-twins](concepts/digital-twins.md) | Virtual representations of physical processes; why mechanistic models are the right foundation; the identifiability problem |
| [rl-for-scientific-discovery](concepts/rl-for-scientific-discovery.md) | RL agents navigating scientific hypothesis spaces; MDP formulation for model/equation discovery; Q-learning, dynamic ε-policy, hierarchical RL, graph grammar, Q-table transfer |
| [engineering-ontologies](concepts/engineering-ontologies.md) | Formal knowledge representation for engineering domains; OntoCAPE; building blocks, grammar, reward rules stored and reasoned over; relevant to El Agente knowledge management |

## Entities
| Page | Summary |
|---|---|
| [uspto-dataset](entities/uspto-dataset.md) | Large-scale patent reaction dataset in SMILES format; standard benchmark for reaction prediction and synthesis planning |
| [dexpi-standard](entities/dexpi-standard.md) | Open standard for P&ID representation as semantic XML; pyDEXPI parses to KG; raw Proteus = 150K tokens → conceptual KG = 7K tokens |
| [taylor-couette-reactor](entities/taylor-couette-reactor.md) | Flow reactor with rotating inner cylinder; tunable from CSTR-like to PFR-like via rpm; used as experimental test case in Heyer et al. 2025 |
