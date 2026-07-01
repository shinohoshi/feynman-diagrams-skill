# Feynman Diagrams Skill for Claude Code

A Claude Code skill for generating, computing, and drawing Feynman diagrams using **FeynArts + FeynCalc** (Mathematica).

## Capabilities

- **Tree-level** diagram generation and amplitude calculation
- **1-loop** QED/EW corrections (vertex corrections, vacuum polarization, box diagrams)
- **Passarino-Veltman** tensor reduction to scalar master integrals (A0, B0, C0, D0)
- **Cross sections** and decay widths with numerical evaluation
- **Automatic Feynman diagram rendering** via FeynArts `Paint[]`
- **K-factor** analysis with 1-loop QED corrections
- **Detailed physics analysis** of every result

## Supported Processes

Bhabha scattering, Drell-Yan, e+e- -> mu+mu-, e+e- -> gamma gamma, e+e- -> W+W-, e+e- -> ZH, gg -> H (loop), H -> gamma gamma (loop), and more.

## Requirements

- [Mathematica](https://www.wolfram.com/mathematica/)
- [FeynCalc](https://feyncalc.github.io/) (auto-loads FeynArts)
- [wolframscript](https://www.wolfram.com/wolframscript/) (for .nb generation)

## Installation

Place `SKILL.md` in your Claude Code skills directory.

**FeynCalc 10.1.0 users:** Create `$UserBaseDirectory/Kernel/init.m` with:
```mathematica
Get["path/to/FeynCalc/FeynArts/FeynArts312.m"];
```
This ensures `<<FeynCalc` properly loads FeynArts.

## Example Output

Opening the generated `.nb` in Mathematica and evaluating step-by-step produces:
1. Tree-level Feynman diagrams (Paint inline)
2. |M|^2 (symbolic expression)
3. 1-loop Feynman diagrams
4. Tensor-reduced amplitudes
5. Numerical cross section table
6. K-factor + NLO cross sections
7. Cross section vs sqrt(s) plot
8. Physics analysis of every result

## Reference

The canonical test notebook computes **e+e- -> mu+mu-** (tree + 1-loop QED) and is available in the companion repository.
