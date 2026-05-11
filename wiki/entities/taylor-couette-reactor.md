# Taylor-Couette Reactor

**What it is**: A flow reactor characterized by a fixed cylindrical outer shell and a rotating inner cylinder, creating an annular gap through which the reaction mixture flows. The rotation generates characteristic vortex patterns (Taylor vortices) that control mixing.

**Manufacturer in this study**: Autichem Ltd. (prototype ribbed Taylor-Couette reactor)  
**Key operating variable**: Inner cylinder rotation speed (rpm) — controls the flow regime

---

## Flow regime and mixing

The rotation speed determines where the reactor falls on the CSTR-to-PFR spectrum:
- **High rpm** (360 rpm): strong Taylor vortices → high mixing → approaches CSTR behavior
- **Low rpm** (0–60 rpm): laminar or weakly vortical → low mixing → approaches PFR behavior

This tunability makes Taylor-Couette reactors valuable for reactions where mixing intensity affects yield, selectivity, or safety. Applications include photocatalytic, electrochemical, and enzymatic reactions.

---

## Usage in this wiki

| Paper | Usage |
|---|---|
| [[sources/heyer2025-rl-mechanistic-models]] | Physical twin for experimental case study; RL workflow generates mechanistic models from outlet concentration data at four operating conditions (varying rpm and solvent) |

---

## Notes on modeling

**Part I (single-compartment, equation generation only)**:
Best model: NRMSE = 2.4%; neglects axial diffusion; third-order kinetics. When mixing speed was reduced, the workflow adjusted the kinetic constant rather than the diffusion term — physically incorrect (kinetics shouldn't depend on mixing). Reveals the identifiability problem: multiple model structures can fit equally well with different physical interpretations.

**Part II (multi-compartment, topology generation)**:
Automatically identified hydrodynamic topology: CSTR cascade + small bypass + large PFR in series. Relative RMSD = 5.4% on RTD data. The agent independently discovers nearly equal CSTR volumes and a bypass flow — matching Richter et al. (2008) literature models for ribbed TCRs without being told to.

At 0 rpm (stationary rotor): structurally different model — single large CSTR + stronger bypass. More CSTR-like than the rotating case, consistent with the loss of Taylor vortices.

Transfer learning (360 rpm → 0 rpm recalibration): pre-trained Q-table improves success rate by 16 percentage points within 30 min.
