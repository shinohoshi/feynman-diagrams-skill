---
name: feynman-diagrams
description: Use when asked to draw Feynman diagrams, compute scattering amplitudes (tree, 1-loop, or multi-loop), or calculate cross sections or decay widths for particle physics. Triggers include Feynman diagrams, 费曼图, scattering, cross section, amplitude, loop, 圈图, 单圈, FeynCalc, FeynArts, or processes such as "e+ e- -> mu+ mu-", "gg -> h" (loop). Requires FeynArts + FeynCalc on Mathematica.
---

# Feynman Diagrams — FeynArts + FeynCalc

## When to Use

Use this skill when the task involves:

- Drawing Feynman diagrams for papers, theses, or talks
- Computing amplitudes, cross sections, or decay widths (tree, 1-loop, multi-loop)
- Setting up FeynArts/FeynCalc workflows in Mathematica, including generating evaluatable `.nb` notebooks

**Not for:** deriving QFT formalism by hand without diagrams, or numerical lattice/phenomenology codes that do not use FeynArts + FeynCalc.

## Prerequisites

- Mathematica 12+ (with `wolframscript` for `.nb` generation)
- FeynCalc and FeynArts installed (see [README.md](README.md) for install commands)
- For FeynCalc 10.1.0: apply the FeynArts auto-load fix below once

## CRITICAL: How to generate a working .nb

**Two methods to generate .nb — use the right one for the content:**

```bash
# Method A: No Chinese → wolframscript -code (fast, inline)
wolframscript -code '
nb = Notebook[{...}];
Export["output.nb", nb]
'

# Method B: With Chinese → write .wls file + wolframscript -f (avoids encoding bugs)
wolframscript -f build_script.wls
```

**Never write `.nb` expression files directly — they won't evaluate.** The .wls file uses Mathematica code with `Cell["code", "Input"]` format. `Export["file.nb", nb]` handles the `\<...\>` conversion.

The notebook structure inside:

```
Cell["Title", "Title"]
Cell["Subtitle text", "Subtitle"]
Cell["Step N: ...", "Section"]           ← section header
Cell["code; code; code;", "Input"]       ← data preparation
Cell["Paint[diags, ...]", "Input"]       ← Paint in its OWN cell, NO trailing ;
Cell["Analysis: what this means...", "Text"]  ← interpret the result
Cell["Step N+1: ...", "Section"]
...
```

## Notebook structure rules

1. **Paint must be alone.** Put `Paint[...]` in its own `"Input"` cell as the ONLY expression, no `;` after it. If Paint shares a cell with `Print[...];` or assignments, its output is suppressed.
2. **Data before Paint.** Assign `diags = InsertFields[...];` in a separate cell BEFORE the Paint cell. The Paint cell references the variable, nothing else.
3. **No PDF/PNG export.** Diagrams and plots display inline. The last expression in a cell renders automatically.
4. **One concept per cell.** Don't put unrelated operations in the same cell.
5. **Analyze every result.** After each major output (diagrams, |M|², cross section table, plot), add a `"Text"` cell with detailed physics analysis covering: (a) what the result means, (b) which channels dominate and why, (c) comparison with known experimental values, (d) notable features and trends. Use `StyleBox["keyword", Bold]` for emphasis in TextData. Never use `Style[]` inside TextData — it silently breaks Export.

## Tree-level workflow

```mathematica
(* DATA cell: generate diagrams *)
topologies = CreateTopologies[0, 2 -> 2, ExcludeTopologies -> {Tadpoles, WFCorrections}];
diags = InsertFields[topologies,
  {F[2, {1}], -F[2, {1}]} -> {F[2, {2}], -F[2, {2}]},
  InsertionLevel -> {Classes}, Model -> "SMQCD"];
Print["Diagrams: ", Length[diags]]
```

```mathematica
(* PAINT cell: render diagrams — NO trailing ; *)
Paint[diags, ColumnsXRows -> {2, 1},
  SheetHeader -> "e+ e- -> mu+ mu- (Tree Level)",
  PaintLevel -> {Classes}, Numbering -> Simple, ImageSize -> {700, 300}]
```

```mathematica
(* COMPUTE cell: amplitude, |M|^2, cross section *)
amps = FCFAConvert[CreateFeynAmp[diags, PreFactor -> 1],
  IncomingMomenta -> {p1, p2}, OutgoingMomenta -> {k1, k2},
  UndoChiralSplittings -> True, ChangeDimension -> 4, SMP -> True, Contract -> True];
Print["Amplitude 1: ", amps[[1]] // Short];
If[Length[amps] > 1, Print["Amplitude 2: ", amps[[2]] // Short]];

ampSq = (Total[amps] . ComplexConjugate[Total[amps]]) //
  FermionSpinSum[#, ExtraFactor -> 1/4] & // DiracSimplify // Simplify;

FCClearScalarProducts[];
SetMandelstam[s, t, u, p1, p2, -k1, -k2,
  SMP["m_e"], SMP["m_e"], SMP["m_mu"], SMP["m_mu"]];
ampSqMassless = Simplify[ampSq /. {SMP["m_e"] -> 0, SMP["m_mu"] -> 0}];
Print["|M|^2 = ", ampSqMassless];
Print["dsigma/dOmega = ", ampSqMassless/(64 Pi^2 s)];

sigmaTotal = Integrate[ampSqMassless/(64 Pi^2 s) * 2 Pi * Sin[theta], {theta, 0, Pi}];
Print["sigma_total = ", sigmaTotal]
```

## 1-loop workflow

Same Paint rule applies. Key differences from tree:

- `CreateTopologies[1, ...]` for 1-loop
- `ChangeDimension -> D` for dimensional regularization
- `LoopMomenta -> {q}` in FCFAConvert
- After extraction: `FCLoopFindTopologies` → `FCLoopTensorReduce` → scalar integrals

```mathematica
(* DATA cell: generate 1-loop diagrams *)
topologies = CreateTopologies[1, 2 -> 2, ExcludeTopologies -> {Tadpoles, WFCorrections}];
diags = InsertFields[topologies,
  {F[2, {1}], -F[2, {1}]} -> {F[2, {2}], -F[2, {2}]},
  InsertionLevel -> {Classes}, Model -> "SMQCD",
  Restrictions -> {NoLightFHCoupling}];
Print["1-loop diagrams: ", Length[diags]]
```

```mathematica
(* PAINT cell: render up to 24 diagrams — NO trailing ; *)
sample = If[Length[diags] <= 24, diags, Take[diags, 24]];
Paint[sample, ColumnsXRows -> {4, Ceiling[Length[sample]/4]},
  SheetHeader -> "e+ e- -> mu+ mu- (1-loop QED)",
  PaintLevel -> {Classes}, Numbering -> Simple,
  ImageSize -> {1100, Ceiling[Length[sample]/4]*120}]
```

```mathematica
(* REDUCE cell: amplitude + tensor reduction *)
amps = FCFAConvert[CreateFeynAmp[diags, PreFactor -> 1],
  IncomingMomenta -> {p1, p2}, OutgoingMomenta -> {k1, k2},
  LoopMomenta -> {q}, UndoChiralSplittings -> True,
  ChangeDimension -> D, SMP -> True, Contract -> True,
  (* FeynArts uses the opposite sign for the electric charge; align with FeynCalc's SMP["e"] *)
  FinalSubstitutions -> {SMP["e"] -> -SMP["e"]}];

{ampTopo, topos} = FCLoopFindTopologies[Total[amps], {q}];
Print["Topologies: ", Length[topos]];
Do[Print[topos[[i]]], {i, Length[topos]}];

ampMapped = FCLoopApplyTopologyMappings[ampTopo, topos];
ampScalar = FCLoopTensorReduce[ampMapped, {q}, topos];
Print["Reduced to scalar integrals:"];
Print[ampScalar // Short];

basis = Cases[ampScalar, GLI[__], Infinity] // Union;
Print["Scalar integral basis: ", basis]
```

## FeynCalc 10.1.0 FeynArts fix

`<<FeynCalc` in v10.1.0 does NOT auto-load FeynArts. Fix once (permanent):

```mathematica
Export[FileNameJoin[{$UserBaseDirectory, "Kernel", "init.m"}],
  "Get[\"C:/Users/.../FeynArts/FeynArts312.m\"];\n", "Text"]
```

Replace the path with the actual FeynCalc installation path. After this, `<<FeynCalc` always loads FeynArts.

## Particle names (FeynArts convention)

| Particle | FeynArts | Particle | FeynArts |
|---|---|---|---|
| e-, mu-, ta- | `F[2, {1}]`, `{2}`, `{3}` | photon | `V[1]` |
| positron, etc. | `-F[2, {1}]` | Z | `V[2]` |
| neutrinos | `F[1, {1..3}]` | W+ / W- | `V[3]` / `-V[3]` |
| quarks u..t | `F[3, {1..6}]` | gluon | `V[5]` |
| Higgs | `S[1]` | Goldstones | `S[2]`, `S[3]` |

Models: `"SM"`, `"SMQCD"`, `"SMEW"`, `"MSSM"`, `"MSSMCT"`.

## Loop toolchain

```
FeynArts → FCFAConvert → FCLoopFindTopologies → FCLoopTensorReduce → scalar integrals (GLI[])
```

For **numerical 1-loop**: `<<FeynHelpers` → `PaXEvaluate[]` (Package-X bridge).  
For **multi-loop IBP**: FeynHelpers → `FIREBurn[]` / `KiraReduce[]` → `PSDIntegrate[]` (pySecDec).

## Common mistakes

| Mistake | Fix |
|---|---|
| Trailing `;` after `Paint[...]` | Put `Paint` alone in its own cell with no semicolon, otherwise the diagrams are not rendered |
| Writing `.nb` as raw expression files | Always generate notebooks via `wolframscript` + `Export["file.nb", ...]` |
| Chinese text inside `wolframscript -code` | Use Method B: write a `.wls` file and run `wolframscript -f` |
| `Style[]` inside `TextData` | Use `StyleBox["text", Bold]` — `Style[]` silently breaks Export |
| Mixing unrelated operations in one cell | One concept per cell: data preparation, Paint, and computation each in their own cell |
| Forgetting `LoopMomenta -> {q}` or `ChangeDimension -> D` at 1-loop | Include both options in `FCFAConvert` |
| FeynArts not loading after `<<FeynCalc` (v10.1.0) | Apply the `init.m` fix once (see above) |
| Painting too many diagrams at once | Take a sample of at most 24 diagrams to keep the layout readable |

## Common processes

```mathematica
(* Bhabha *)       {F[2,{1}],-F[2,{1}]} -> {F[2,{1}],-F[2,{1}]}
(* ee->mumu *)     {F[2,{1}],-F[2,{1}]} -> {F[2,{2}],-F[2,{2}]}
(* ee->gaga *)     {F[2,{1}],-F[2,{1}]} -> {V[1],V[1]}
(* ee->WW *)       {F[2,{1}],-F[2,{1}]} -> {-V[3],V[3]}
(* ee->ZH *)       {F[2,{1}],-F[2,{1}]} -> {V[2],S[1]}
(* gg->ttbar *)    {V[5],V[5]} -> {F[3,{6}],-F[3,{6}]}
(* gg->H (loop) *) {V[5],V[5]} -> {S[1]}           (* CreateTopologies[1,...] *)
(* H->gaga (lp) *) {S[1]} -> {V[1],V[1]}           (* CreateTopologies[1,...] *)
```

## Reference

For full physics coverage, notebook structure, and tool citations, see [README.md](README.md) in this repository.
