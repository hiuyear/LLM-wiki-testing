# Digital Twins

**What it is**: A virtual representation of a physical system that mirrors its behavior in real time (or near real time). The "twin" runs as a computational model; the physical system is the "real twin." Data flows from physical → digital (sensor readings, concentration measurements, etc.) to keep the model calibrated; the digital twin can then be used for simulation, optimization, and prediction.

In chemical engineering: a digital twin of a reactor lets you simulate the effect of changing flow rates, temperatures, or feed compositions *before* running the physical experiment — accelerating process development and reducing waste.

---

## Why mechanistic models are the right foundation

A digital twin can be implemented as a neural network, a lookup table, or a mechanistic model. The choice matters enormously:

| Model type | Extrapolation | Interpretability | Update cost | Physical validity |
|---|---|---|---|---|
| Neural network (black box) | Poor | None | Requires retraining | Not guaranteed |
| Symbolic regression | Limited | Medium | Refit from scratch | Not guaranteed |
| Mechanistic (first-principles) | Good | Full | Targeted parameter re-estimation | Guaranteed by construction |

Mechanistic models are built from differential balance equations (mass, energy, momentum). Every term corresponds to a physical phenomenon. This means:
- The model can be validated against domain knowledge, not just fit quality
- Small geometry changes require updating parameters, not relearning the whole structure
- The model generalizes to operating conditions not seen during fitting

The tradeoff: mechanistic model development is slow and requires expertise. This is the motivation for automating it (see [[sources/heyer2025-rl-mechanistic-models]]).

---

## Mechanistic model structure (Heyer et al. 2025)

The most general form of a reactor model starts from:

```
0 = ∂ψ/∂t + ∇·ψv + ∇·ϕ_ψ − σ_ψ^P − σ_ψ^F
```

Where ψ represents the intensive quantity (density, enthalpy, entropy, velocity). The five terms are: accumulation, convective transport, diffusive transport, production, supply.

The modeling task is to:
1. Decide which terms are negligible for this specific system
2. Substitute each retained term with appropriate constitutive equations (Fick's law, Arrhenius, etc.)
3. Estimate unknown parameters from experimental data

Each such decision is a modeling assumption. The audit trail of decisions *is* the model's interpretability.

---

## The identifiability problem

A critical issue in digital twin development: **multiple model structures can fit the same data**. In Heyer et al. (2025), reducing mixer speed caused the RL workflow to lower the kinetic constant — but domain knowledge says kinetics are independent of mixing. The correct adjustment should have been to the diffusion term. The fit was equally good either way.

This is structural non-identifiability: from the output data alone, you can't distinguish between two physically different explanations. Mitigation requires:
- Structural identifiability analysis before parameter estimation
- Additional experimental data (e.g., residence time distributions) that distinguish the hypotheses
- Domain knowledge as a prior in the reward function (reward shaping)

---

## Compartmentalization

A single-compartment model (one balance volume for the whole reactor) can't capture non-ideal flow patterns: vortices, bypasses, dead zones. The model topology — how many compartments, how they connect — must be discovered from data alongside the equations.

Part II (Laub et al. 2026) automates this. Model topologies are represented as colored digraphs and discovered by an upper-level RL agent using graph grammar rules. The result for the Taylor-Couette reactor was a topology (CSTR cascade + bypass + PFR) that matches expert-built literature models at ~5% RMSD.

## Transfer learning for recalibration

When operating conditions change, the model needs structural recalibration (not just parameter re-fitting). Laub et al. (2026) show that initializing the RL agent with the Q-table from a prior modeling campaign at different conditions speeds up recalibration by ~16 percentage points in success rate within 30 min.

Why: the Q-table encodes which graph manipulations led to good and bad topologies. This knowledge is partially transferable even when the correct new topology is structurally different from the old one — the agent starts by knowing what *not* to try.

---

## Papers
- [[sources/heyer2025-rl-mechanistic-models]] — RL-automated mechanistic model generation for reactor digital twins; Part I: equation generation
- [[sources/laub2026-rl-mechanistic-models-ii]] — Part II: compartmentalization, graph grammar, reward shaping, transfer learning
