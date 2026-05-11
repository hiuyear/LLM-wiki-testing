# Automated Generation of Mechanistic Models for Chemical Process Digital Twins Using RL (Part II)

**Authors**: Jan-Frederic Laub, Jiyizhe Zhang, Mathis Heyer, Alexei A. Lapkin  
**Institution**: Dept. of Chemical Engineering and Biotechnology, University of Cambridge; Process Systems Engineering, RWTH Aachen  
**Journal**: Computers and Chemical Engineering 204 (2026) 109384  
**DOI**: 10.1016/j.compchemeng.2025.109384  
**Note**: Direct continuation of [[sources/heyer2025-rl-mechanistic-models]] (Part I). Same group, same reactor, extends the workflow with the compartmentalization module.

---

## What Part I left unfinished

Part I (Heyer et al. 2025) automated equation generation — given a fixed compartmentalization (how many balance volumes, how they connect), an RL agent decides which terms to keep/neglect/substitute. But the compartmentalization itself was fixed as a single compartment. That's a major constraint: real reactors have non-ideal flow (vortices, bypasses, dead zones) that a single-compartment model can't capture.

Part II automates compartmentalization: finding the right model *topology* (graph of compartments and interfaces) from data — before equation generation even runs.

---

## Approach

### Model topology as a colored digraph

A reactor model topology is a directed graph where:
- **Vertices** = compartments (e.g., CSTR, PFR, mixing zone)
- **Edges** = interfaces (e.g., convective flow, diffusive mass transfer, bypass)
- **Colors** = building block types — the class of equations that compartment/interface inherits

This graph structure is the state in the upper-level MDP. Two topologies are considered equal if they are graph-isomorphic (same structure, same colors) — canonicalized using the `scott` algorithm.

### Upper-level MDP: topology generation

| Component | Instantiation |
|---|---|
| State | Colored digraph representing current model topology |
| Action | Graph production rule (insert compartment, connect interface, add bypass) |
| Reward | R = R₁ (knowledge-shaped) + R₂ (accuracy-based after parameter estimation) |
| Terminal | Not fixed — episodes run up to n_max steps |

**Graph grammar**: Manipulation rules are formalized as *productions* — rewrite rules of the form `L ::= R` (if subgraph L appears, replace it with R). This standardizes how topologies evolve and constrains the search space to physically meaningful graphs. Different grammars can be stored for different unit operation types.

**Reward splitting**:
- R₁ (cheap): knowledge-shaped — penalizes topologies missing known phenomena (e.g., no diffusive mass transfer in a PTC reactor). Computed from graph structure alone. Only models passing R₁ threshold proceed to R₂.
- R₂ (expensive): accuracy-based — embed parameter estimation optimization, compute fit error, map to [0,1]. Computed only for topologies that passed R₁.

This two-stage reward cuts computation by skipping parameter estimation for topologically unreasonable models.

### Hierarchical integration

Upper-level (topology) → Lower-level (equation generation, from Part I). For each topology candidate, the equation generator runs multiple times. The best-fit result from the lower level feeds back as R₂ to the upper level. The two RL agents operate simultaneously, each learning its own Q-table.

### Ontology for knowledge management

All workflow artifacts — building blocks, graph grammar rules, reward shaping functions, generated model topologies, constitutive equations, parameter values — are stored in an OntoCAPE-based ontology (OntoCAPE is a standard process systems engineering ontology). Two subdomains:
- **Process subdomain**: physical reality — reactor conditions, phenomena, operating states
- **Modeling subdomain**: virtual realm — building blocks, topologies, equations, parameters

This decouples knowledge from specific modeling campaigns. A model generated in one campaign can be retrieved and continued in the next — including by the RL agents themselves (transfer learning).

---

## Key Results

### Case study 1: Phase transfer catalysis (in silico)

Multi-phase system (organic + aqueous phases, catalyst diffusing across FILM interface). Reward space is highly sparse — most topologies are equally bad. Key findings:

- **Random search with reward shaping finds the true model faster than RL without reward shaping** — in sparse spaces, domain knowledge is more valuable than learning
- RL with reward shaping + moderate exploitation achieves the best performance
- Reward shaping works by eliminating bad topologies from parameter estimation (cheap structural filter before expensive simulation)

### Case study 2: Taylor-Couette reactor (experimental)

Two levels of data: residence time distribution (RTD) for hydrodynamics; reaction yield for chemistry.

**Hydrodynamic model** (RTD data only, topology search with equation generator bypassed):
- Automatically identified topology: CSTR cascade + bypass + PFR in series
- Relative RMSD = 5.4% — within the same range as expert-built literature models
- The agent correctly infers CSTR volumes are nearly equal (without being told), and identifies a small bypass — matching the Richter et al. (2008) approach for ribbed TCRs
- Found in episode 9 after 177 candidate models, ~20 min runtime

**Integrated model** (RTD → hydrodynamics → reaction yield):
- Topology from RTD step passed to equation generator with reaction-appropriate building blocks
- Two candidate models found (first-order vs second-order kinetics); both fit well; RTD error comparable to Part I diffusion-based models
- True reaction order undetermined — would need additional experiments to discriminate

**Transfer learning (model recalibration)**:
- Scenario: TCR runs at 360 rpm → power fault → drops to 0 rpm → need new model
- Strategy A: run workflow from scratch (empty Q-table)
- Strategy B: initialize with Q-table from 360 rpm model generation run
- Result: Strategy B finds a recalibrated model (R₂ ≥ 0.9) **16 percentage points more often** within 30 min
- Even though the 360 rpm and 0 rpm models are structurally different (vortex-dominated vs bypass-dominated), pre-training helps navigate away from dead ends faster

---

## Key conceptual contributions

**Reward shaping as domain knowledge injection**: Instead of hoping the agent discovers that mass transfer is necessary, you tell it via a penalty. This converts sparse rewards into denser, navigable landscapes. The lesson: when the search space is combinatorially large and reward-sparse, domain knowledge in the reward function is more valuable than sophisticated RL architecture.

**Q-table as transferable memory**: The Q-table accumulates structured knowledge about which graph manipulations tend to lead toward good models. When conditions change, this knowledge isn't wasted — the agent starts knowing what *doesn't* work and can jump to more promising regions. This is the RL equivalent of warm-starting.

**Ontology as knowledge persistence layer**: The ontology isn't just a database; it's the mechanism by which the workflow accumulates institutional knowledge across modeling campaigns. An expert curates the ontology (building blocks, grammar, reward schemes); the RL agents use it. Role of humans shifts from modeler → knowledge curator.

---

## Limitations acknowledged

1. Computational time: minutes (not seconds) for recalibration — not yet real-time
2. Tabular Q-learning: doesn't scale to large graphs; graph neural networks would be needed
3. No uncertainty quantification module (acknowledged as future work)
4. Not embedded in a full digital twin framework (no sensor integration, no live KPIs)

---

## Relevance to El Agente Gráfico

The ontology-based knowledge management pattern is directly applicable:
- Building blocks (reusable templates for agent capabilities)
- Grammar-constrained action spaces (not all agent actions are valid at all states)
- Reward shaping from domain knowledge (don't let the agent explore physically impossible configurations)
- Q-table transfer as a form of agent memory across sessions

---

## Cross-references
- Concept: [[concepts/digital-twins]] (extends with transfer learning and compartmentalization)
- Concept: [[concepts/rl-for-scientific-discovery]] (extends with hierarchical RL, graph grammar, Q-table transfer)
- Concept: [[concepts/engineering-ontologies]]
- Entity: [[entities/taylor-couette-reactor]] (RTD results + hydrodynamic modeling)
- Related source: [[sources/heyer2025-rl-mechanistic-models]] (Part I — equation generation module)
