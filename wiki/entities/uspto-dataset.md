# USPTO Reaction Dataset

**What it is**: The United States Patent and Trademark Office reaction dataset — a large-scale collection of chemical reactions extracted from US patent literature. One of the most widely used benchmarks in computational chemistry and reaction prediction.

**Format**: Reactions encoded as SMILES strings (reactants, reagents, products)  
**Scale**: Millions of reactions in full; studies typically sample subsets  
**Preprocessing**: Often cleaned using ORDerly (removes duplicates, canonicalizes SMILES, filters rare entries)

---

## Usage in this wiki

| Paper | Usage |
|---|---|
| [[sources/bunkova2026-kg-llm-synthesis]] | 50k reactions sampled, loaded into Neo4j as a bipartite KG for Text2Cypher evaluation |

---

## Notes

- SMILES canonicalization matters: the same molecule can be written many ways in SMILES. Canonicalization ensures each molecule has exactly one representation, which is critical for graph-based identity matching.
- Bunkova et al. filter to reactions with ≤4 reactants, ≤4 products, ≤4 agents, ≤4 solvents — covers ~95% of the dataset.
- Preprocessed via the ORDerly library (Wigh et al., J. Chem. Inf. Model. 2024).
