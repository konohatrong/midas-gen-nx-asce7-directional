# Wind Load Generator — ASCE 7-22 (MWFRS, Directional)

Generate and assign **ASCE 7-22** Main Wind Force Resisting System (Directional
Procedure) wind loads on **MIDAS GEN NX** building frames — directly from the model.

## What it does
- **Geometry:** regular building frames with non-uniform bays.
  Roof types: **gable**, **unequal-pitch gable** (two slopes), and **monoslope**.
- **ASCE 7-22:** velocity pressure (Kz, qz), Eq 27.3-1 (Kd in the load equation),
  wall + roof Cp (per-plane angle), GCpi (±), all four wind directions, low-slope
  zoned roofs, stepped windward wall, gable-end wind posts, corner columns.
- **Workflow — assign to the model open in GEN NX:**
  1. **Connect** with your MAPI-Key.
  2. **Create tagging groups** and assign your wall/roof elements to them in GEN NX
     (`WWcol, LWcol, WWroof, LWroof, ROOF, ENDWALL0, ENDWALL1, CORNER`); existing
     project groups are never deleted.
  3. **Refresh** so the tool re-reads the tagged model.
  4. **Preview** the ASCE pressures, then **Apply** to write the loads.
  The tool auto-detects gable vs monoslope and unequal-pitch, handles meshed
  members, and assigns loads **only to tagged elements**.
- **Output:** per-direction load cases (`Wind ±X/±Y · ±GCpi`), load groups, and
  `/db/BMLD` beam loads — zoned/trapezoidal distributions mapped across meshed
  members.

## Units
Reads the model's units (`/db/UNIT`) and converts loads to match. Inputs in US
(mph, ft, lb/ft²) or SI (m/s, m, N/m²).

## Disclaimer
This plug-in is provided **as-is**. It has been **developed and verified only by the
author's own tests** (validated against the ASCE 7-22 Wind Loads Guide, Example
G3-1). It does **not** replace professional engineering judgment or independent
review by a licensed engineer. The user is solely responsible for checking all
inputs and results before using them in design. No warranty is given and the
authors accept no liability for its use.

Design and Verify by: **Peerapat Pinyopojanee, M. Eng** · Powered and Coded by: **Claude AI**
