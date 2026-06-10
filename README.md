# Wind Load Generator — MIDAS GEN NX plug-in (publish repo)

ASCE 7-22 MWFRS (Directional) wind load generator that runs **inside MIDAS GEN NX**
as a Marketplace plug-in. This repo holds the **release artifacts only**; the
source/engine lives in the development repo (`midas_API`: engine in
`pilot project/wind load generator/`, plug-in bundle in
`To midas/wind-load-generator/source/`).

**Design and Verify by : Peerapat Pinyopojanee, M. Eng**
**Powered and Coded by : Claude AI**

## Contents
| Path | What |
|---|---|
| `releases/YYYY-MM-DD_vX.Y.Z/` | **one folder per version**, named by its first publish date — the version history (rebuilds of the same version reuse their folder) |
| `releases/…/wind-load-generator-vX.Y.Z.zip` | **Upload this** on MIDAS Marketplace → MyWork |
| `releases/…/…-no-manifest.zip` | fallback if the host rejects `manifest.json` |
| `validation/` | V&V evidence: `VALIDATION.md` + live campaign run logs |

The plug-in **source** is not duplicated here — it lives only in the dev repo
(`To midas/wind-load-generator/source/`); the zips are the release artifact.

## Features (v2.0.0)
- Connect → create/refresh tagging Structure Groups (non-destructive) → Preview →
  Apply, on the model open in GEN NX (tagged elements only)
- Roof shapes auto-detected: gable, unequal-pitch gable, monoslope; meshed
  (subdivided) members handled; any plan position/rotation via ridge azimuth
- Load sets: **Wind only** (±X/±Y × ±GCpi) · **Wind + MWFRS** (56 cases) ·
  **Full** (Dead + Roof Live + wind + 48 MWFRS C1–C4 = 58 cases, canonical order)
- Windward-wall q_z zoning, per-surface selection, non-destructive apply
  (Dead/Live/EQ and other wind cases preserved), chunked pushes
- **⬇ Calc report (PDF)** — step-by-step ASCE 7-22 derivation with
  self-validating CHECK columns (internal-pressure constancy; MWFRS base-shear
  ratios = Fig 27.3-8 factors)
- Engine = the same validated Python compute as the desktop tool, run in-browser
  via Pyodide (bundled; no internet needed beyond the MIDAS relay)

## Load case reference — all 58 cases, one by one

This is the **Full set** in the exact canonical order the tool pushes to MIDAS
(`/db/STLD` case numbers 1–58 on a clean apply). The **Wind + MWFRS** set is
cases 3–58 (56 cases); **Wind only** is cases 3–10.

### How to read a case name
| Token | Meaning |
|---|---|
| `+X / -X` | wind blowing toward +X / −X (normal to the ridge for a default frame) — for asymmetric or tapered buildings −X is NOT a mirror of +X, so both are generated |
| `+Y / -Y` | wind blowing toward +Y / −Y (parallel to the ridge, onto a gable end) |
| `+GCpi` | positive internal pressure (building pressurized inside): pushes **outward** on every surface — adds to roof uplift, relieves inward wall pressure |
| `-GCpi` | negative internal pressure (internal suction): pulls **inward** — increases inward wall pressure, relieves roof uplift |
| `e+ / e-` | torsional eccentricity sign: the lateral resultant is shifted ±0.15·B **transverse** to the wind (for ±X wind: along Y, B = building length; for ±Y wind: along X, B = span). Implemented as linear-graded frame line loads, so base shear AND base torsion are reproduced without a rigid diaphragm |
| `+X+Y` etc. | diagonal case: both principal axes loaded **simultaneously** |

MIDAS load types: Dead → `D`, Roof Live → `L`, all wind/MWFRS → `W`.

### ASCE 7-22 Fig 27.3-8 — the four MWFRS design cases
| Case | Pressure factor | What it checks |
|---|---|---|
| **Case 1** | 100% | full design pressure on each principal axis separately — the basic strength case |
| **Case 2** | 75% + torsion M_T (e = ±0.15·B) | torsional response from non-uniform pressure / accidental eccentricity |
| **Case 3** | 75% on BOTH axes simultaneously | diagonal (corner) wind; roof = 100% of the larger Case-1 roof per Fig 27.3-8 **Note 2** (not summed, not factored) |
| **Case 4** | 56.25% (=0.75²) on both axes + torsion | combined diagonal wind + torsion; roof = envelope at 75% per Note 2 |

### Cases 1–2 · Roof gravity (Full set only)
| # | Load case | What it combines |
|---|---|---|
| 1 | `Dead` | user Dead pressure × tributary width, downward (−GZ) line loads on every roof member, acting **along the slope** (true roof area) |
| 2 | `Roof Live` | user Roof Live pressure × tributary, downward on the **horizontal projection** (plan area) of the roof |

### Cases 3–10 · Directional wind (Eq 27.3-1, full pressures)
Each = windward wall q_z (zoned if enabled) + leeward wall, side walls, and roof
Cp pressures for that direction, combined with one internal-pressure sign.
| # | Load case | What it combines |
|---|---|---|
| 3 | `Wind +X +GCpi` | wind toward +X: +X wall is windward (Cp 0.8, q_z), −X wall leeward, roof windward/leeward (or zoned) slopes for +X; internal pressure pushing outward |
| 4 | `Wind +X -GCpi` | same external +X pressures; internal suction pulling inward |
| 5 | `Wind -X +GCpi` | wind toward −X: roles swap (−X wall windward, +X leeward; roof slopes swap — exact for unequal-pitch roofs); internal outward |
| 6 | `Wind -X -GCpi` | same −X externals; internal suction |
| 7 | `Wind +Y +GCpi` | wind along the ridge toward +Y: first gable end windward, far end leeward, long walls are side walls (Cp −0.7), roof zoned by distance from the windward gable end (bounds h, 2h); internal outward |
| 8 | `Wind +Y -GCpi` | same +Y externals; internal suction |
| 9 | `Wind -Y +GCpi` | wind toward −Y (last gable end windward — differs from +Y on tapered plans); internal outward |
| 10 | `Wind -Y -GCpi` | same −Y externals; internal suction |

### Cases 11–18 · MWFRS Case 1 (factor 1.00 — each axis separately)
Each is exactly the matching `Wind` case at 100% — kept as separate cases so the
MWFRS set is complete and self-contained in load combinations.
| # | Load case | What it combines |
|---|---|---|
| 11 | `MWFRS C1 +X +GCpi` | 100% of case 3 (full +X design pressures, internal outward) |
| 12 | `MWFRS C1 +X -GCpi` | 100% of case 4 |
| 13 | `MWFRS C1 -X +GCpi` | 100% of case 5 |
| 14 | `MWFRS C1 -X -GCpi` | 100% of case 6 |
| 15 | `MWFRS C1 +Y +GCpi` | 100% of case 7 |
| 16 | `MWFRS C1 +Y -GCpi` | 100% of case 8 |
| 17 | `MWFRS C1 -Y +GCpi` | 100% of case 9 |
| 18 | `MWFRS C1 -Y -GCpi` | 100% of case 10 |

### Cases 19–34 · MWFRS Case 2 (75% + torsion, e = ±0.15·B)
75% of the directional pressures, with the **horizontal** loads linearly graded
along the building so the lateral resultant shifts by the eccentricity (torsion
M_T = 0.75·P·e); vertical roof loads carry the 0.75 factor only.
| # | Load case | What it combines |
|---|---|---|
| 19 | `MWFRS C2 +X e+ +GCpi` | 75% of +X pressures + resultant shifted +0.15·B(length) toward +Y; internal outward |
| 20 | `MWFRS C2 +X e+ -GCpi` | as 19, internal suction |
| 21 | `MWFRS C2 +X e- +GCpi` | 75% of +X + shift −0.15·B toward −Y; internal outward |
| 22 | `MWFRS C2 +X e- -GCpi` | as 21, internal suction |
| 23 | `MWFRS C2 -X e+ +GCpi` | 75% of −X + shift +0.15·B(length); internal outward |
| 24 | `MWFRS C2 -X e+ -GCpi` | as 23, internal suction |
| 25 | `MWFRS C2 -X e- +GCpi` | 75% of −X + shift −0.15·B(length); internal outward |
| 26 | `MWFRS C2 -X e- -GCpi` | as 25, internal suction |
| 27 | `MWFRS C2 +Y e+ +GCpi` | 75% of +Y + resultant shifted +0.15·B(span) toward +X; internal outward |
| 28 | `MWFRS C2 +Y e+ -GCpi` | as 27, internal suction |
| 29 | `MWFRS C2 +Y e- +GCpi` | 75% of +Y + shift −0.15·B(span) toward −X; internal outward |
| 30 | `MWFRS C2 +Y e- -GCpi` | as 29, internal suction |
| 31 | `MWFRS C2 -Y e+ +GCpi` | 75% of −Y + shift +0.15·B(span); internal outward |
| 32 | `MWFRS C2 -Y e+ -GCpi` | as 31, internal suction |
| 33 | `MWFRS C2 -Y e- +GCpi` | 75% of −Y + shift −0.15·B(span); internal outward |
| 34 | `MWFRS C2 -Y e- -GCpi` | as 33, internal suction |

### Cases 35–42 · MWFRS Case 3 (diagonal: 75% on both axes)
Walls of BOTH directions at 75% applied together; roof = 100% of the **larger**
Case-1 roof of the two directions (Fig 27.3-8 Note 2).
| # | Load case | What it combines |
|---|---|---|
| 35 | `MWFRS C3 +X+Y +GCpi` | 75% +X walls + 75% +Y walls together; envelope roof; internal outward |
| 36 | `MWFRS C3 +X+Y -GCpi` | as 35, internal suction |
| 37 | `MWFRS C3 +X-Y +GCpi` | 75% +X walls + 75% −Y walls; envelope roof; internal outward |
| 38 | `MWFRS C3 +X-Y -GCpi` | as 37, internal suction |
| 39 | `MWFRS C3 -X+Y +GCpi` | 75% −X walls + 75% +Y walls; envelope roof; internal outward |
| 40 | `MWFRS C3 -X+Y -GCpi` | as 39, internal suction |
| 41 | `MWFRS C3 -X-Y +GCpi` | 75% −X walls + 75% −Y walls; envelope roof; internal outward |
| 42 | `MWFRS C3 -X-Y -GCpi` | as 41, internal suction |

### Cases 43–58 · MWFRS Case 4 (diagonal 56.25% + torsion)
Walls of both directions at 56.25% with the SAME-sign eccentricity applied on
each axis simultaneously; roof = envelope at 75% (= 0.75 × Case-1) per Note 2.
| # | Load case | What it combines |
|---|---|---|
| 43 | `MWFRS C4 +X+Y e+ +GCpi` | 56.25% +X & +Y walls, both resultants shifted +0.15·B on their transverse axes; envelope roof; internal outward |
| 44 | `MWFRS C4 +X+Y e+ -GCpi` | as 43, internal suction |
| 45 | `MWFRS C4 +X+Y e- +GCpi` | 56.25% +X & +Y walls, both shifted −0.15·B; internal outward |
| 46 | `MWFRS C4 +X+Y e- -GCpi` | as 45, internal suction |
| 47 | `MWFRS C4 +X-Y e+ +GCpi` | 56.25% +X & −Y walls, shifts +0.15·B; internal outward |
| 48 | `MWFRS C4 +X-Y e+ -GCpi` | as 47, internal suction |
| 49 | `MWFRS C4 +X-Y e- +GCpi` | 56.25% +X & −Y walls, shifts −0.15·B; internal outward |
| 50 | `MWFRS C4 +X-Y e- -GCpi` | as 49, internal suction |
| 51 | `MWFRS C4 -X+Y e+ +GCpi` | 56.25% −X & +Y walls, shifts +0.15·B; internal outward |
| 52 | `MWFRS C4 -X+Y e+ -GCpi` | as 51, internal suction |
| 53 | `MWFRS C4 -X+Y e- +GCpi` | 56.25% −X & +Y walls, shifts −0.15·B; internal outward |
| 54 | `MWFRS C4 -X+Y e- -GCpi` | as 53, internal suction |
| 55 | `MWFRS C4 -X-Y e+ +GCpi` | 56.25% −X & −Y walls, shifts +0.15·B; internal outward |
| 56 | `MWFRS C4 -X-Y e+ -GCpi` | as 55, internal suction |
| 57 | `MWFRS C4 -X-Y e- +GCpi` | 56.25% −X & −Y walls, shifts −0.15·B; internal outward |
| 58 | `MWFRS C4 -X-Y e- -GCpi` | as 57, internal suction |

**Count check:** 2 gravity + 8 wind + C1 (4 dirs × 2 GCpi = 8) + C2 (4 × 2e × 2
= 16) + C3 (4 diagonals × 2 = 8) + C4 (4 × 2e × 2 = 16) = **58**. For a building
oriented at any plan angle (ridge azimuth ≠ 90°), every load above is decomposed
into world GX + GY components — the case set and names stay identical.

**Validation built in:** the ⬇ Calc report (PDF) checks every MWFRS case's base
shear against its factor (1.0 / 0.75 / 0.5625 × the matching full-wind case) and
prints a green OK / red MISMATCH per case — see Step 5 of the report.

## Validation
See [`validation/VALIDATION.md`](validation/VALIDATION.md): ASCE 7-22 Guide
Example G3-1; an independent low-slope hand calc; and the live oriented-matrix
campaign **104/104 PASS** (13 scenarios × 8 placements, geometry round-trip +
world-frame load vectors) — log in `validation/runs/`.

## Release workflow
1. Develop & test in the dev repo (`run_local_check.bat` must pass).
2. `python build_plugin.py` → rebuilds the bundle, the versioned zips, and
   **syncs this folder** (`plugin/`, zips, `validation/`).
3. Update `CHANGELOG.md`, bump `VERSION` in `build_plugin.py` for the next cycle.
4. Commit/push THIS folder's repo; upload the zip on MIDAS → MyWork.

## Disclaimer
Provided **as-is**; developed and verified only by the author's own tests.
Independent review by a licensed engineer is required before production use.
