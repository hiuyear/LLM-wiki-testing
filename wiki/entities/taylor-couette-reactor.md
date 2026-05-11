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

The best RL-generated model (NRMSE = 2.4%) for the base case neglected axial diffusion and used third-order kinetics. When mixing speed was reduced, the workflow adjusted the kinetic constant rather than the diffusion term — physically incorrect (kinetics shouldn't depend on mixing). This highlights the identifiability problem: without structural identifiability analysis, multiple model structures can fit equally well while having different physical interpretations.

Literature suggests multi-zonal compartmental models (not single-compartment) are more accurate for Taylor-Couette reactors. The compartmentalization module in Part II of the study (Laub et al. 2025) is expected to address this.
