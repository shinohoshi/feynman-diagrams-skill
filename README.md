# Feynman Diagrams Skill for Claude Code

Generates, computes, and analyzes Feynman diagrams for particle physics using **FeynArts + FeynCalc** on Mathematica. Handles tree-level, 1-loop, and multi-loop calculations across QED, QCD, and electroweak interactions.

## Quick Start

1. Install `SKILL.md` in your Claude Code skills directory
2. Ask Claude: *"Compute e+e- -> mu+mu- at tree level + 1-loop QED"*
3. Claude generates a `.nb` notebook — open in Mathematica, evaluate step-by-step
4. FeynArts draws the diagrams inline, FeynCalc computes amplitudes and cross sections

## Physics Coverage

| Interaction | Processes |
|---|---|
| **QED** | e+e- → μ+μ-, e+e- → γγ, Bhabha scattering (e+e- → e+e-), Compton, Moller |
| **Weak (charged current)** | μ → e ν ν̄, e+e- → W+W-, single-W production |
| **Weak (neutral current)** | e+e- → Z → ff̄ (any fermion pair), νν̄ production |
| **Higgs** | e+e- → ZH (Higgsstrahlung), gg → H (top loop), H → γγ, H → ZZ\*, H → WW\* |
| **QCD** | gg → tt̄, qq̄ → tt̄, gg → gg, qq̄ → gg, gg → jets |
| **Electroweak (full)** | Any SM 2→2 process with γ, W, Z exchange + interference |
| **Loop-only** | gg → H, H → γγ, gg → ZH, γγ → WW (all zero at tree level in SM) |

**Models:** Standard Model (`SM`, `SMQCD`, `SMEW`), MSSM (`MSSM`, `MSSMCT`), and **custom UFO models** for BSM physics (e.g., extended Higgs sectors for phase transition studies).

## Computational Pipeline

```
FeynArts                FeynCalc                  FeynHelpers (optional)
──────────              ────────                  ──────────────────────
CreateTopologies   →   FCFAConvert            →   PaXEvaluate (Package-X)
InsertFields       →   FermionSpinSum         →   FIREBurn / KiraReduce
CreateFeynAmp      →   DiracSimplify          →   PSDIntegrate (pySecDec)
Paint (diagrams)   →   FCLoopFindTopologies   →   QGCreateAmp (QGRAF)
                        FCLoopTensorReduce
                        SetMandelstam
                        Integrate (cross section)
```

**Tree level:** diagrams → amplitudes → |M|² → dσ/dΩ → σ_total  
**1-loop:** diagrams (D-dim) → topology identification → PV tensor reduction → scalar master integrals (A₀, B₀, C₀, D₀) → numerical evaluation

## Notebook Output

Every generated `.nb` contains:

| Step | Content |
|---|---|
| 1 | Load FeynCalc (with init.m auto-loading FeynArts) |
| 2 | **FeynArts Paint[]** — Feynman diagrams rendered inline |
| 3 | **FeynCalc** — symbolic amplitude and \|M\|² |
| 4 | **1-loop diagrams** — vertex corrections, vacuum polarization, box diagrams |
| 5 | **Tensor reduction** — topology identification, Passarino-Veltman to scalar integrals |
| 6 | **Numerical cross sections** — table at benchmark energies (Belle II, LEP, ILC, CLIC) |
| 7 | **K-factor** — 1-loop QED/EW NLO/LO ratio with decomposition |
| 8 | **Cross section plot** — LO vs NLO with resonance markers |
| Each | **Physics analysis** — interpretation of every result |

## Requirements

- [Mathematica](https://www.wolfram.com/mathematica/) (version 12+)
- [FeynCalc](https://feyncalc.github.io/) — `Import["https://raw.githubusercontent.com/FeynCalc/feyncalc/master/install.m"]; InstallFeynCalc[]`
- [wolframscript](https://www.wolfram.com/wolframscript/) (bundled with Mathematica, used internally for `.nb` generation)

**FeynCalc 10.1.0 note:** `<<FeynCalc` may not auto-load FeynArts. Permanent fix — create `$UserBaseDirectory/Kernel/init.m`:

```mathematica
Get["C:/Users/<name>/AppData/Roaming/Mathematica/Applications/FeynCalc/FeynArts/FeynArts312.m"];
```

## Key Design Decisions

- **Paint in its own cell** — never put `;` after `Paint[]` or combine it with `Print[]`; the semicolon suppresses diagram rendering
- **wolframscript Export** — `.nb` files must be generated via `wolframscript Export[]`, not written as raw expression files
- **StyleBox in TextData** — use `StyleBox["text", Bold]` for bold in analysis cells; `Style[]` silently breaks Export
- **One concept per cell** — data preparation, Paint rendering, and computation are separate cells for clean step-by-step evaluation

## Related Skills

This skill is designed to work alongside:
- **Cosmo Phase Transition** — cosmological phase transition calculations using CosmoTransitions
- Both are part of the [Claude Physics Skills](https://github.com/shinohoshi) suite for theoretical physics research

## References

- [FeynCalc Manual](https://feyncalc.github.io/FeynCalcBook/)
- [FeynArts Documentation](https://feynarts.de/)
- [CosmoTransitions](https://github.com/clwainwright/CosmoTransitions)
