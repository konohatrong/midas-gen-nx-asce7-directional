# Wind Load Generator — MIDAS GEN NX plug-in (publish repo)

ASCE 7-22 MWFRS (Directional) wind load generator that runs **inside MIDAS GEN NX**
as a Marketplace plug-in. This repo holds the **release artifacts only**; the
source/engine lives in the development repo (`midas_API`, folder
`pilot project/wind load generator/`).

**Design and Verify by : Peerapat Pinyopojanee, M. Eng**
**Powered and Coded by : Claude AI**

## Contents
| Path | What |
|---|---|
| `releases/YYYY-MM-DD_vX.Y.Z/` | one folder per release (publish date + version) holding the upload zips — the version history |
| `releases/…/wind-load-generator-vX.Y.Z.zip` | **Upload this** on MIDAS Marketplace → MyWork |
| `releases/…/…-no-manifest.zip` | fallback if the host rejects `manifest.json` |
| `plugin/` | the CURRENT upload file set, unzipped (for inspection/diff) |
| `validation/` | V&V evidence: `VALIDATION.md` + live campaign run logs |

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
