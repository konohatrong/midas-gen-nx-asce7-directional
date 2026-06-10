# Changelog — Wind Load Generator plug-in

## v2.1.0 — 2026-06-11
**Unified UI** — the plug-in and the desktop tool are now generated from ONE
shared UI source (same form, same handlers; only the backend adapter differs:
FastAPI on desktop, MAPI+Pyodide in the plug-in). Feature drift between the
local tool and the packed plug-in is structurally impossible from this version.
- Plug-in UI gains the desktop's step-by-step layout, mode notes, and richer
  status reporting; capability gating hides desktop-only paths (generate,
  MWFRS-only) instead of forking the page.
- No engineering changes: same engine, same 58-case set, same calc report
  (232 offline tests pass; engine identical to the 104/104-validated v2.0.0).

## v2.0.0 — 2026-06-11
First Marketplace-ready release.
- **Fixed: plug-in icon + load failure.** Root cause: `manifest.json` must follow
  the official `@midasit-dev/cra-template-moaui` schema exactly
  (`short_name`/`name`, `icons → favicon.ico` with `"64x64 32x32 24x24 16x16"`
  `image/x-icon`, `start_url "."`, `display standalone`, theme/background colors,
  and the MIDAS-specific `width`/`height` panel size). Earlier custom-schema
  manifests made the plug-in fail to load; shipping no manifest left the
  `ol-icon-image` placeholder. `favicon.ico` is generated and linked from `<head>`.
- Pyodide loaded by a static `<script>` tag (template-style), bundled locally;
  CDN only as fallback. jsPDF bundled locally for the in-plug-in PDF report.
- Load sets: Wind only / **Wind + MWFRS (56)** / Full (+ Dead/Roof Live, 58).
- **Calc report (PDF)**: step-by-step ASCE 7-22 derivation with CHECK columns.
- Ridge azimuth (any orientation), windward-wall q_z zoning, per-surface
  selection, non-destructive apply, chunked BMLD pushes.
- Engine parity with the desktop tool (230+ offline tests; oriented-matrix live
  campaign 104/104 PASS).
