# Wiki Log

## [2026-05-11] ingest | Goldstein et al. 2025 — pyDEXPI

**Source**: `raw/LAPSE-2025.0371-1v1.pdf`  
**Pages created**:
- `sources/goldstein2025-pydexpi.md` (new)

**Pages updated**: `entities/dexpi-standard.md` (full pyDEXPI implementation section: Pydantic classes, 473 types, attribute_category system, toolkits), `index.md`, `log.md`

**Cross-paper note**: Same TU Delft group as ChatP&ID. pyDEXPI is the shared infrastructure layer. This paper explains the 150K→7K token compression ratio from Alimin et al. — the Proteus XML contains geometry/graphics data from all 473 classes; the conceptual KG strips everything non-process. No new concepts needed; dexpi-standard.md substantially upgraded.

---

## [2026-05-11] ingest | Laub et al. 2026 — RL for Mechanistic Reactor Models (Part II)

**Source**: `raw/1-s2.0-S0098135425003874-main.pdf`  
**Pages created**:
- `sources/laub2026-rl-mechanistic-models-ii.md` (new)
- `concepts/engineering-ontologies.md` (new)

**Pages updated**: `concepts/digital-twins.md` (compartmentalization + transfer learning), `concepts/rl-for-scientific-discovery.md` (hierarchical RL, graph grammar, Q-table transfer, sparse rewards), `entities/taylor-couette-reactor.md` (Part II RTD results), `index.md`, `log.md`

**Cross-paper note**: Direct continuation of Heyer et al. 2025 (Part I). Key new insight: in sparse reward spaces, domain knowledge in the reward function beats sophisticated RL architecture. The ontology pattern is the most transferable idea to El Agente.

---

## [2026-05-11] ingest | Heyer et al. 2025 — RL for Mechanistic Reactor Models (Part I)

**Source**: `raw/1-s2.0-S0098135425002832-main.pdf`  
**Pages created**:
- `sources/heyer2025-rl-mechanistic-models.md` (new)
- `concepts/digital-twins.md` (new)
- `concepts/rl-for-scientific-discovery.md` (new)
- `entities/taylor-couette-reactor.md` (new)

**Pages updated**: `index.md`, `log.md`

**Cross-paper note**: Different domain from papers 1–2 (mechanistic modeling vs. LLM/KG). Lapkin group (Cambridge) is adjacent to the Matter Lab ecosystem. Key transferable concept: MDP formulation for structured agent decision-making — more reliable than open-ended LLM planning when the action space is well-defined and enumerable.

---

## [2026-05-11] ingest | Alimin & Schweidtmann 2026 — ChatP&ID / GraphRAG for P&IDs

**Source**: `raw/2603.22528v1.pdf`  
**Pages created**:
- `sources/alimin2026-chatp-id.md` (new)
- `concepts/graphrag.md` (new)
- `concepts/agentic-systems.md` (new)
- `entities/dexpi-standard.md` (new)

**Pages updated**: `concepts/kg-grounding-vs-rag.md` (added GraphRAG section + alimin cross-ref), `concepts/text2cypher.md` (added CypherRAG subsection + alimin cross-ref), `index.md`

**Cross-paper note**: Same group as Bunkova et al. (TU Delft, Schweidtmann). Same stack (Neo4j, LangGraph). Both papers independently find LLM-as-judge > semantic similarity for factual evaluation.

---

## [2026-05-11] ingest | Bunkova et al. 2026 — KG-LLM Synthesis Retrieval

**Source**: `raw/2601.16038v1.pdf`  
**Pages created**:
- `sources/bunkova2026-kg-llm-synthesis.md` (new)
- `concepts/text2cypher.md` (new)
- `concepts/kg-grounding-vs-rag.md` (new)
- `concepts/chain-of-verification.md` (new)
- `entities/uspto-dataset.md` (new)

**Pages updated**: `index.md`
