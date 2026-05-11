# Reinforcement Learning for Scientific Discovery

**What it is**: Using RL agents to navigate large, structured search spaces in scientific domains — equation spaces, reaction networks, process flowsheets — where the goal is to find a configuration that optimizes some measurable objective (accuracy, efficiency, cost).

The core pattern: formulate scientific discovery as a Markov Decision Process. The agent explores the hypothesis space, receives reward based on how well the hypothesis fits data, and learns which regions of the space are worth exploring.

---

## Why RL fits certain scientific problems

RL is well-suited when:
1. **The search space is too large for exhaustive enumeration** — combinatorial explosion makes brute force infeasible
2. **The reward is only available at the end of a trajectory** — you can't evaluate a partial model; you need the full model to compute fit
3. **Actions have variable cardinality** — the number of valid choices depends on the current state (e.g., how many constitutive equations exist for a given variable)
4. **Decisions are sequential and interdependent** — early choices constrain later ones

These conditions hold for mechanistic model generation (Heyer et al. 2025), process flowsheet design (Khan & Lapkin 2020), and kinetic model discovery.

---

## MDP Formulation for Mechanistic Model Generation (Heyer et al. 2025)

| MDP Component | Concrete instantiation |
|---|---|
| **State S** | System of differential equations + decision status of each term (decided/undecided) |
| **Action A** | {Neglect, Substitute known value, Declare parameter, Substitute constitutive equation} |
| **Reward R** | `1 / (1 + SSE)` mapping fit error to [0,1]; optional complexity penalty |
| **Transition T** | Deterministic (same action in same state always leads to same next state) |
| **Terminal state** | All variables decided → model is executable |

State representation: postfix sequence of Python objects. Parsed to Pyomo model for simulation. Reward computed after parameter estimation on experimental data.

---

## Q-Learning Specifics

**Algorithm**: tabular Q-learning, Monte Carlo update style. Q(s,a) is the average reward over all training trajectories that passed through (s,a).

**Exploration-exploitation**:
- Static ε: probability ε of random action, (1−ε) of greedy. Optimum at ε=0.7 for the in silico case.
- Dynamic ε: ε decreases linearly from 1→0. Finds true model in 80% of runs within 317 iterations vs. 500+ for static ε. Exploration early → exploitation late is the right schedule.

**Why tabular (not neural)?**: The state space is structured and discrete (postfix sequences of objects). A lookup table over seen (s,a) pairs is simpler and doesn't require vector representations. Neural Q-learning would require encoding the equation sequence as a fixed-size vector — harder and unnecessary here.

**Complexity penalty**: Adding a term that rewards simpler models slows identification when the true model is complex. Useful as a regularizer when the goal is parsimony, but actively harmful when searching for the objectively correct model.

---

## RL vs. Alternatives

| Method | Search space | Physical priors | Generalization |
|---|---|---|---|
| Exhaustive enumeration | Full | Yes (constrained) | N/A |
| Symbolic regression | Continuous equation space | No | Poor (overfits noisy data) |
| RL (mechanistic) | First-principles constrained | Yes (conservation laws) | Good |
| LLM-based planning | Open-ended | Only via prompt | Variable |

Key advantage over symbolic regression: RL operates *within* the space of physically valid models (the constitutive equation database enforces this). Symbolic regression searches a broader space and can find accurate-but-unphysical models that fail on unseen conditions.

---

## Hierarchical RL

For large search spaces, flat RL struggles with reward sparsity (the reward only comes at the end of a long trajectory). Hierarchical RL decomposes this:
- **Upper-level agent**: decides on coarse structure (model topology — which compartments and interfaces)
- **Lower-level agent**: specifies details within each compartment (equation generation from Part I)

In Laub et al. (2026), the upper-level agent manipulates a colored digraph representing the reactor topology using graph grammar production rules. For each topology candidate, the lower-level agent runs several times; the best-fit result feeds back as reward to the upper level. The two Q-tables are independent and learned simultaneously.

## Graph grammar for constrained search

Instead of allowing arbitrary graph manipulations, **graph grammar** formalizes valid actions as production rules: `L ::= R` means "if subgraph L exists, you may replace it with R." This:
- Constrains the search to physically meaningful topologies (no disconnected graphs, no isolated compartments)
- Can be stored in an ontology and reused across modeling campaigns
- Allows different grammars for different unit operation types

## Q-table transfer (transfer learning)

The Q-table is a record of which (state, action) pairs have historically led to good models. When operating conditions change and the model needs recalibration, the new run can be initialized with the old Q-table. Result (Laub et al. 2026): ~16 percentage point improvement in finding a recalibrated model within 30 min, even when the correct new topology is structurally different from the old one.

The mechanism: the old Q-table tells the agent which graph manipulations are *rarely* productive (low Q-values). These are still probably unproductive in the new scenario, so the agent skips them and explores more promising directions faster.

## Sparse rewards and domain knowledge

In some search spaces, the reward landscape is nearly flat: most model topologies are equally bad. In this case, RL without domain knowledge performs no better than random search. The solution is **reward shaping**: inject domain knowledge (e.g., "a PTC reactor *must* have a diffusive mass transfer interface") as a cheap preliminary reward R₁ that filters out topologies before the expensive parameter estimation step (R₂).

Lesson: sophisticated RL architecture is wasted on sparse reward spaces without domain knowledge injection. Fix the reward landscape first.

---

## Papers
- [[sources/heyer2025-rl-mechanistic-models]] — RL for reactor mechanistic model generation; Q-learning, MDP formulation, dynamic ε-policy
- [[sources/laub2026-rl-mechanistic-models-ii]] — hierarchical RL for compartmentalization + equation generation; graph grammar; reward shaping; Q-table transfer learning
