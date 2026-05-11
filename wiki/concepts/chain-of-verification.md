# Chain-of-Verification (CoVe)

**What it is**: A self-correction framework where an LLM generates a response, then a validator checks it against a checklist, and a corrector applies targeted fixes. The loop repeats until the response passes or a maximum number of attempts is reached.

General pattern:
1. Generator LLM produces candidate output
2. Validator LLM checks against a fixed checklist → outputs list of specific errors
3. Corrector LLM applies minimal edits to resolve flagged issues
4. Repeat up to N times

---

## Application in Text2Cypher (Bunkova et al. 2026)

Used to fix LLM-generated Cypher queries before execution:
1. LLM generates candidate Cypher query
2. Query is checked for executability with `EXPLAIN` in Neo4j
3. If not executable: LLM corrector fixes based on error message (SMILES masked to avoid special-char issues)
4. LLM validator checks against task-specific checklist
5. If invalid: LLM corrector applies minimal edits
6. Max 3 attempts

**What CoVe fixed**: Mainly *completeness* errors — missing reaction components (reactants, products, agents). In zero-shot settings, Reactants/Products missing errors dropped ~80%.

**What CoVe failed to fix**: Task-specific failures like duplicate molecules or incomplete entity names. The validator (generic checklist) missed 86–95% of these errors. The corrector was not the bottleneck — the validator was.

---

## Key insight

Generic validators don't generalize. A checklist built from past error analysis works well for the errors it was built on, but fails on novel error types. For CoVe to be effective, the validator needs to be **task-specific** and **schema-aware** — knowing the exact structure of valid outputs for each query type.

---

## Verdict for chemistry agent design

CoVe is worth using in zero-shot settings as a cheap safety net. In one-shot settings with good exemplars, the marginal benefit is small — invest in better exemplar selection instead. If deploying CoVe, build the validator from schema and task structure, not just from generic error lists.

---

## Papers
- [[sources/bunkova2026-kg-llm-synthesis]] — CoVe applied to Text2Cypher over reaction KGs; validator identified as bottleneck
