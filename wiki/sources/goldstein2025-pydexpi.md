# Goldstein et al. 2025 — pyDEXPI: A Python Framework for P&IDs

**Full title**: pyDEXPI: A Python framework for piping and instrumentation diagrams using the DEXPI information model  
**Authors**: Dominik P. Goldstein, Lukas Schulze Balhorn, Achmad Anggawirya Alimin, Artur M. Schweidtmann  
**Affiliation**: Process Intelligence Research Group, TU Delft (same group as ChatP&ID)  
**Venue**: ESCAPE 35, Ghent, Belgium, July 2025  
**Published in**: Systems and Control Transactions 4:1365-1370 (2025)  
**DOI**: 10.69997/sct.139043 | LAPSE:2025.0371  
**Code**: [github.com/process-intelligence-research/pyDEXPI](https://github.com/process-intelligence-research/pyDEXPI)

---

## Problem

No open-source implementation of the DEXPI standard existed before pyDEXPI. The only prior tool was P&ID Verificator (commercial, closed-source, no API). This created two barriers:

1. **Interoperability**: each vendor rolled their own DEXPI parser, incompatible with each other
2. **GenAI adoption**: without a Python-native DEXPI library, integrating P&IDs into LLM pipelines required bespoke parsing for every project

The gap was especially acute because DEXPI is now the industry-standard format and academic groups (including the Schweidtmann group themselves) kept reimplementing the same XML parsing.

---

## Implementation

### Core design: Pydantic data classes

pyDEXPI implements all 473 DEXPI data classes as Python Pydantic models, organized into 11 packages:

| Package | Contents |
|---|---|
| DexpiModel | Root container |
| MetaData | Revision history, revisions |
| PlantStructure | Plant items, tag prefixes, nodes |
| Equipment | Vessels, heat exchangers, pumps, etc. |
| Piping | Pipes, fittings, nozzles |
| Instrumentation | Instruments, actuators, signal lines |
| Customization | Custom classes, references |
| Enumerations | Typed enum values |
| PhysicalQuantities | Engineering units, measured values |
| Graphics | Shape, position, label data |
| DataTypes | Shared primitive types |

### Inheritance mapping

The DEXPI specification uses a type hierarchy. pyDEXPI maps this faithfully:

- **Supertype → class** relationship becomes Python class inheritance
- **Subtype → inverse** (child class inherits from parent class)
- **Compositional attributes** (e.g., a vessel *contains* nozzles) → `attribute_category="compositional"` field annotation
- **Reference attributes** (e.g., a valve *references* its actuator) → `attribute_category="reference"`
- **Data attributes** (e.g., nominal diameter) → `attribute_category="data"`

Every class inherits from `DexpiBaseModel`, which assigns a UUID on instantiation and generates a new UUID on copy (ensuring uniqueness across model instances). Primitive data type classes inherit from `DexpiDataTypeBaseModel` instead.

Enumerations (e.g., valve types, signal types) are implemented as Python `Enum` subclasses. Basic data types (strings, integers) are replaced with Python builtins rather than wrapping them in classes.

---

## Functionality

**Import**: Parse Proteus XML (DEXPI's canonical format) into a pyDEXPI object model  
**Save/load**: Pickle serialization for caching parsed models  
**Export**: Convert to NetworkX graph for downstream analytics and KG construction

**Toolkits** (higher-level helpers on top of the base classes):
- `model_toolkit` — whole-model operations (traversal, statistics)
- `piping_toolkit` — pipe routing and connectivity queries
- `instrumentation_toolkit` — instrument loop identification

---

## Why this paper matters

pyDEXPI is the *infrastructure layer* that makes ChatP&ID and similar systems possible. Alimin et al. 2026 (ChatP&ID) used pyDEXPI to parse smart P&IDs into Neo4j — but pyDEXPI was unpublished at the time. This paper formally documents the design decisions.

The Pydantic-based approach is worth noting: Pydantic handles validation, serialization, and type-checking automatically. It's a pragmatic choice — the DEXPI spec is complex (473 classes), and Pydantic keeps the implementation honest by enforcing type constraints at runtime.

---

## Downstream uses at time of publication

| Project | Usage |
|---|---|
| ChatP&ID (Alimin et al. 2026) | Parse smart P&IDs → conceptual KG → 4-tool GraphRAG |
| P&ID autocorrection (Schulze Balhorn et al.) | Structural validation and error detection |
| Computer vision digitization | Convert image-based P&IDs to DEXPI via CV → then pyDEXPI |

---

## Cross-paper notes

- Same TU Delft group (Schweidtmann) as Alimin et al. 2026. pyDEXPI is the shared infrastructure; ChatP&ID is one downstream use.
- The 473-class implementation explains the 150K→7K token compression ratio in ChatP&ID: Proteus XML contains all 473 class instances including geometry/graphics data; the conceptual KG strips everything except process-relevant structure.
- The GitHub URL is live and actively maintained as of this writing.
