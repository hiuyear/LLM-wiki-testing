# DEXPI Standard

**What it is**: Data Exchange in the Process Industries — an open, vendor-neutral standard for representing Piping and Instrumentation Diagrams (P&IDs) in a machine-readable semantic format. Published as ISO 15926-based XML (Proteus format).

**Maintained by**: DEXPI e.V. (industry consortium including BASF, Bayer, Evonik, Siemens, ABB, and others)  
**Format**: XML (Proteus schema) — stores equipment, piping, instrumentation, control logic, and safety elements as structured objects with typed relationships

---

## Why it matters for LLM integration

P&IDs are the central engineering blueprints of chemical plants. Every pipe, valve, sensor, and controller appears in a P&ID. Before DEXPI, P&IDs were stored as static images or CAD files — no machine-readable structure.

DEXPI makes P&IDs *queryable*. But raw DEXPI XML is massive:
- Up to **150K tokens per page** (one P&ID page)
- Semantically noisy: URIs, UUIDs, internal metadata dominate the token count
- LLMs — even large ones — struggle with raw Proteus files at this scale

The solution (from ChatP&ID): parse DEXPI via pyDEXPI, then collapse to a conceptual-level knowledge graph (~7K tokens). This is the practical path to LLM integration.

---

## pyDEXPI

Open-source Python library from the Process Intelligence Research Group at TU Delft (Schweidtmann group). First and only open-source DEXPI implementation — no prior tool had a Python API or open license.

**Source**: [github.com/process-intelligence-research/pyDEXPI](https://github.com/process-intelligence-research/pyDEXPI)  
**Paper**: [[sources/goldstein2025-pydexpi]]

### Implementation details

473 DEXPI data classes implemented as **Pydantic Python data classes** across 11 packages (DexpiModel, MetaData, PlantStructure, Equipment, Piping, Instrumentation, Customization, Enumerations, PhysicalQuantities, Graphics, DataTypes).

Key design decisions:
- **Supertype → class inheritance** (DEXPI type hierarchy maps directly to Python class hierarchy)
- **Attribute category annotations**: `attribute_category="compositional"` for containment relationships, `"reference"` for cross-entity references, `"data"` for scalar properties
- **`DexpiBaseModel`**: base class for all component classes — assigns UUID on instantiation, new UUID on copy
- **`DexpiDataTypeBaseModel`**: base class for primitive types; Python builtins (str, int) replace simple DEXPI data types
- **Enumerations**: DEXPI enumerations → Python `Enum` subclasses

Pydantic handles validation and type-checking automatically at runtime — critical for a 473-class spec.

### Functionality

- **Import**: parse Proteus XML into pyDEXPI object model
- **Save/load**: pickle serialization for caching
- **Export**: convert to NetworkX graph
- **Toolkits**: `model_toolkit` (whole-model traversal), `piping_toolkit` (pipe routing/connectivity), `instrumentation_toolkit` (instrument loop identification)

Each P&ID component → graph node; relationships (compositional, reference) → directed edges.

---

## Abstraction Levels (via pyDEXPI → Neo4j)

| Level | Description | ~Token count |
|---|---|---|
| Complete | One-to-one mapping from DEXPI (full hierarchy, noisy URIs) | ~150K |
| Process | Collapses piping segments; equipment is primary node type | ~50K |
| Conceptual | Further collapses instrumentation; clean human-readable labels | ~7K |

The conceptual level achieves 85% cost reduction vs raw Proteus with +5% accuracy — the tradeoff between detail and LLM-friendliness strongly favors abstraction at current model capabilities.

---

## Usage in this wiki

| Paper | Usage |
|---|---|
| [[sources/goldstein2025-pydexpi]] | Formally documents the pyDEXPI implementation: Pydantic classes, 473 types, import/export/toolkit architecture |
| [[sources/alimin2026-chatp-id]] | pyDEXPI used to parse smart P&IDs into Neo4j; conceptual-level KG fed to 4-tool GraphRAG system |

---

## Notes

- DEXPI is relevant beyond P&IDs: the broader DEXPI data model covers equipment datasheets, valve specs, and instrument loop diagrams. The P&ID subset is the most commonly implemented.
- "Smart P&IDs" = P&IDs saved in DEXPI format. Legacy P&IDs are typically raster images (JPG/PDF) — DEXPI doesn't help there without a prior digitization step.
- The PyDEXPI library is relatively young and maintained by an academic group. Production use may require additional hardening.
