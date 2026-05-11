# Zhang et al. 2026 — Semantic Framework for Chemical Process Digitalisation

**Full title**: A semantic framework for chemical process digitalisation using ontologies  
**Authors**: Shuyuan Zhang, Yong Ren Tan, Cuong Manh Nguyen, Dogancan Karan, Srishti Ganguly, Nicholas Jose, Mei Qi Lim, Shuqiao Guo, Lianlian Jiang, Markus Kraft, Alexei A. Lapkin  
**Affiliations**: Cambridge CARES (Singapore), University of Cambridge, A*STAR I2R (Singapore), Accelerated Materials Ltd  
**Journal**: Chemical Engineering Journal 533 (2026) 174361  
**DOI**: 10.1016/j.cej.2026.174361

---

## Problem

Chemical process digitalisation faces a fundamental integration problem: sensor data, physical models, and AI models live in separate silos with incompatible formats. Connecting them requires a structured, machine-readable common layer. Without one, digital twins remain one-off implementations that can't interoperate or be reused.

The paper proposes: use ontologies as the backbone. Represent both the physics (model ontology) and the plant structure (process ontology) in a knowledge graph — then write agents that navigate the graph to automatically build, calibrate, and deploy models.

---

## Architecture: Three Layers

**Layer 1 — Ontology definition**: Two custom ontologies define the schema:

| Ontology | Covers | Key classes |
|---|---|---|
| **Model ontology** | Physics / first-principles knowledge | Law (MathML), Parameter (with bounds), (Metric) Unit |
| **Process ontology** | Plant structure | Unit Operation, Stream, Pipe Manifold, Process Variable |

The bridge: `Process Variable` in the process ontology is linked to `Parameter` in the model ontology. This link is what lets agents automatically match plant sensor tags to model parameters.

**Layer 2 — Semantic representation**: Instances of both ontologies + trained AI model form the digital twin. The process ontology is a graph of the actual plant (visualised in Neo4j: unit operations = red nodes, streams = blue, pipe manifolds = green, process variables = yellow).

**Layer 3 — Contextual agents**: Four agents automate the full modelling pipeline:

| Agent | Function |
|---|---|
| Subprocess identification agent | Traverses KG to find which physical laws apply to which subprocesses |
| Model construction agent | Retrieves laws from model ontology, converts MathML → Python ODE via MathML2Code |
| Model calibration agent | Fits parameters against process data; uses SciPy differential evolution |
| Model application agent | Executes calibrated physical + AI models for online monitoring |

---

## The MathML trick

Physical laws are stored in the model ontology as **MathML** — a standard for encoding mathematical structure and relationships. A `MathML2Code` module converts them to runnable Python. This means the model ontology is not just documentation: it is executable. Adding a new law = adding a MathML node to the KG. The agents pick it up automatically.

---

## Cross-domain ontology stack

The process ontology doesn't try to define everything from scratch. It imports and links to established ontologies:

| Ontology | Used for |
|---|---|
| OntoCAPE | Chemical process engineering concepts |
| OntoDevice | Physical device representation (flow meters, pumps, etc.) |
| DEXPI | Unit operations and streams — compatibility with P&ID standards |
| OM (Ontology of Measure) | Units and quantities |
| SAREF | Smart device functions |
| OntoSpecies | Chemical species |
| OntoBMS | Building management (control logic) |

This is the emerging standard ontology stack for digital process engineering. No single ontology covers everything; interoperability requires linking across all of them.

---

## System infrastructure

Two virtual machines (EdgeVM, AgentVM) host agents and software. Cloud databases:

| Database | Purpose |
|---|---|
| Gremlin GraphDB | Ontology / KG storage |
| MongoDB | Time-series sensor data |
| PostgreSQL | Structured metadata, intermediate results |

Data pipeline: Kepware OPC-UA server (plant) → AMPLE Client → PostgreSQL → Azure IoT Edge → cloud. Keycloak handles authentication; Nginx handles certificate validation.

---

## Case studies

**Case 1 — Tennessee Eastman (TE) process** (in silico benchmark):
- 5 unit operations, 22 process variables, 15 fault types
- Physical model: mass balance + gas-liquid phase transition balance (Clausius-Clapeyron)
- Subprocess identification agent correctly identifies which units each balance applies to
- Result: ontology-based model is more robust than statistical baselines; fewer false alarms for controllable faults that the control system handles automatically

**Case 2 — AMPLE pilot plant dosing module** (real deployment):
- 4 storage tanks, 3 operational modes (rinse / prime / reaction), 54 sensor parameters
- Physical model (mass balance) + Autoencoder (AE) data-driven model run in parallel
- AE trained on 28,946 normal samples; alarm threshold at 99th percentile of reconstruction error
- 3 simulated fault types tested: tank leaking, flow rate offset, valve mismatch
- Physical model: slower response (uses time window), more robust, adapts to minor system changes
- AE: faster, more sensitive, covers wider fault patterns — but needs training data

Combined approach outperforms either alone.

---

## Acknowledged limitations

- Manual ontology creation is the primary bottleneck. The paper explicitly flags this and suggests future AI automation (extracting ontology instances from process diagrams)
- Finite set of physical laws in model ontology; needs expansion for tasks beyond anomaly detection
- Compatibility with diverse sensor vendors / cloud architectures requires further work

---

## Cross-paper notes

- **Same Lapkin group** as Heyer et al. 2025 and Laub et al. 2026. Those papers automate model *generation* (finding the right equations); this paper automates model *management and deployment* (storing, connecting, and executing the models). Complementary: one generates the model, the other hosts it.
- OntoCAPE appears here and in Laub et al. 2026 — it's the shared foundation ontology for chemical process engineering across both groups.
- DEXPI appears as one of the cross-domain ontologies. Same DEXPI as the TU Delft P&ID papers — further evidence that DEXPI is becoming the process industry's interoperability standard.
- Nicholas Jose (Accelerated Materials Ltd, co-author) is in the autonomous chemistry space — adjacent to El Agente context.
- Markus Kraft (Cambridge) has a background in The World Avatar project, a long-running effort to build a global KG of physical-world knowledge using ontologies. This paper is in that tradition.
