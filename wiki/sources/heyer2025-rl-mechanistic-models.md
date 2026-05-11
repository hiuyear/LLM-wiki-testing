# Automated Generation of Mechanistic Models for Chemical Process Digital Twins Using RL (Part I)

**Authors**: Mathis Heyer, Jiyizhe Zhang, Naoto Sugisawa, Jan-Frederic Laub, Alexei A. Lapkin  
**Institution**: Dept. of Chemical Engineering and Biotechnology, University of Cambridge; Process Systems Engineering, RWTH Aachen  
**Journal**: Computers and Chemical Engineering 202 (2025) 109281  
**DOI**: 10.1016/j.compchemeng.2025.109281  
**Note**: Part I of a 2-part study. Part II (Laub et al. 2025, ChemRxiv) covers the compartmentalization module and learning-based recalibration. Lapkin is also PI at the Cambridge Centre for Advanced Research in Singapore — closely adjacent to the Matter Lab ecosystem.

---

## Problem

Developing mechanistic models for chemical reactor digital twins is:
- **Specialized**: requires skilled modelers who understand differential balance equations, constitutive relationships, and solver behavior
- **Slow**: formulate → simulate → compare → revise → repeat
- **Not scalable**: current tools (ASPEN, gPROMS) are expensive, designed for human operators, not automated pipelines

The goal: automate the reactor model generation process — given experimental concentration data from a reactor, produce an accurate, interpretable mechanistic model without human modeling expertise.

**Why mechanistic, not data-driven?** Mechanistic models:
- Are derived from conservation principles (mass, energy, momentum) — physically valid by construction
- Extrapolate beyond training data
- Are interpretable: each term corresponds to a physical assumption
- Can be updated incrementally as new data arrives

Data-driven (neural network) models achieve none of these. Symbolic regression is faster but ignores physics and overfits on sparse data.

---

## Approach: RL-Driven Model Generation

### The key insight: model derivation as sequential decision-making

Starting from a general differential mass balance:

```
0 = ∂ψ/∂t + ∇·ψv + ∇·ϕ_ψ − σ_ψ^P − σ_ψ^F
```

A human modeler makes a sequence of decisions for each term/variable:
1. **Neglect** — this phenomenon doesn't matter here (e.g., radiative heat in a liquid reactor)
2. **Substitute known value** — this quantity was measured (e.g., inlet flow rate)
3. **Declare as parameter** — unknown, will be estimated from data (e.g., diffusion coefficient)
4. **Substitute constitutive equation** — pull from the equation library (e.g., Fick's law, Arrhenius)

This is a Markov Decision Process: state = system of equations with decisions made so far; action = one of the four choices above; reward = model fit quality after parameter estimation.

### Reinforcement learning agent

- **Algorithm**: tabular Q-learning (Monte Carlo update — average reward over all trajectories through each state-action pair)
- **Exploration policy**: dynamic ε — starts at ε=1 (fully random), decreases linearly to ε=0 (fully greedy). This outperforms static ε strategies.
- **Reward function**: squashing function `1 / (1 + SSE)` mapping error to [0, 1]; optional complexity penalty term discourages unnecessarily complex models
- **Unsolvable model penalty**: reward = 0.1 (not 0) to prevent gradient shock during training

### State representation: postfix notation

Equations are stored as sequences of Python objects in postfix notation. Why postfix?
- Unambiguous (no parentheses needed)
- Equal operator marks equation boundaries naturally → multiple equations concatenated into one sequence
- Can be parsed back to executable Pyomo code via reverse shunting yard algorithm
- Agent steps through symbolic elements in order, taking one action per undecided term

### Ontological equation database

Domain knowledge (constitutive equations like Fick's law, Arrhenius, power-law kinetics) is stored in a structured, queryable Python dictionary. When the agent selects "substitute with constitutive equation," it queries this database for candidates. The new terms introduced by the constitutive equation are appended to the state — the agent must then decide on those too.

### Seven-module workflow

Input → Compartmentalization → **Equation Generation (RL agent)** → Test → Solver + Parameter Estimation → Post-processing

This paper focuses on the Equation Generation module. Compartmentalization (how to split the reactor volume) is addressed in Part II.

---

## Key Results

### In silico case study (synthetic data from a 1D dispersion model)

- Workflow **correctly reconstructs** the true model (NRMSE = 2.6×10⁻¹¹)
- RL agent finds optimal model with **80% probability in 317 iterations** (~5 min on a laptop)
- **1.5× speedup** vs. exhaustive enumeration in 37% fewer iterations
- Robust to Gaussian noise up to ~10% (above typical lab measurement uncertainty)
- Dynamic ε outperforms all static ε strategies
- Complexity penalty slows identification of the true model when the true model is more complex than alternatives with comparable accuracy

### Experimental case study (Taylor-Couette reactor, esterification)

- Best model: NRMSE = 2.4%; neglects axial diffusion; third-order kinetics with k_p = 1.52×10⁻⁵ m⁶ mol⁻² s⁻¹
- Four models with NRMSE ≤ 13.3% — all physically reasonable approximations
- Workflow correctly adapts to changed operating conditions (different mixing speeds, different solvent)

**Critical failure mode found**: When mixing speed decreases, the workflow adjusts the *kinetic constant* to match the slower reaction progress — but domain knowledge says mixing affects *diffusion*, not kinetics. The kinetic constant should be independent of mixing. This reveals a fundamental identifiability issue: multiple model structures can fit the same data. The workflow doesn't currently perform structural identifiability analysis before fitting.

---

## Conceptual Contributions

**vs. symbolic regression**: RL-generated models start from first-principles balance equations — they obey conservation laws by construction. Symbolic regression finds any equation that fits the data, ignoring physical priors. RL models also extrapolate more reliably, and small data perturbations don't produce wildly different model structures (a known failure mode of symbolic regression).

**Interpretability as a design goal**: Every modeling decision the RL agent makes corresponds to a physical assumption (neglect diffusion, assume first-order kinetics, etc.). The workflow outputs a full audit trail of assumptions alongside the model.

---

## Relevance to El Agente Gráfico

Different domain (mechanistic reactor modeling, not graphical interface) but several transferable patterns:

1. **Ontological database as structured domain knowledge**: the equation database is exactly the pattern El Agente needs for organizing chemical knowledge — not a flat knowledge graph, but a schema-driven, queryable store of domain entities
2. **MDP formulation for agent decision-making**: structured decision spaces with well-defined actions, rewards, and terminal states are more reliable than open-ended LLM planning. Useful pattern when the decision space is enumerable
3. **Interpretability as a requirement**: the paper treats interpretability as a hard design constraint (item #1 of six design principles), not an afterthought. Same philosophy should apply to El Agente — every agent action should be explainable
4. **Identifiability as a pre-condition to fitting**: the structural identifiability gap in this paper (multiple models fit the same data) is analogous to the hallucination problem in LLM-based agents — models that look correct but aren't. Need validation before trusting outputs

---

## Cross-references
- Concept: [[concepts/digital-twins]]
- Concept: [[concepts/mechanistic-modeling]]
- Concept: [[concepts/rl-for-scientific-discovery]]
- Entity: [[entities/taylor-couette-reactor]]
