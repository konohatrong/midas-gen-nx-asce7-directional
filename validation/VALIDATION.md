# Validation — ASCE 7-22

The compute core is validated against **two** ASCE 7-22 cases that together cover
both roof-Cp regimes for wind normal to ridge:

| # | Case | Roof regime exercised | Source | Test |
|---|---|---|---|---|
| 1 | G3-1 gable, θ=18.4° | **uniform** windward roof (θ≥10°) + zoned parallel | official Guide | `test_validation_g3_1.py` |
| 2 | low-slope gable, θ=5° | **zoned** windward roof (θ<10°, by distance) | independent hand calc | `test_validation_lowslope.py` |

---

# Example G3-1

The compute core is validated against **ASCE/SEI "Wind Loads: Guide to the Wind
Load Provisions of ASCE 7-22", Example G3-1** — *Commercial/Warehouse Metal
Building*, an enclosed gable low-rise MWFRS Directional example. This is the
Phase 4 gate. Test: [`../tests/test_validation_g3_1.py`](../tests/test_validation_g3_1.py)
(8 tests, tolerance **0.3 psf** — the Guide rounds to 0.1).

## Building (Guide Table G3-1)
- Plan 200 ft × 250 ft, **eave 20 ft**, **roof 4:12 (θ = 18.43°)**
- Rigid frames span 200 ft at **25 ft o.c.** (11 frames)
- **Exposure C**, **V = 115 mph**, K_zt = K_e = 1.0, **K_d = G = 0.85**
- **Enclosed** → GC_pi = ±0.18 · Units **US**

## What matched (engine vs Guide)

| Quantity | Guide | Engine |
|---|---|---|
| Ridge height | 53.3 ft | ✅ |
| Mean roof height h | 36.7 ft | ✅ |
| h/L | 0.18 | ✅ |
| q_h | 34.6 psf | ✅ |
| q_z (0–15 ft) | 28.8 psf | ✅ |
| K_z at h | 1.02 | ✅ |
| Wall C_p | 0.80 / −0.50 (normal) / −0.45 (parallel) / −0.70 | ✅ |
| Roof C_p normal (θ=18.4°) | WW −0.36 & 0.14, LW −0.57 | ✅ |
| Roof C_p parallel | −0.9 / −0.5 / −0.3 (by zone) | ✅ |

### Net design pressures, normal to ridge (Table G3-6), psf — [+GC_pi, −GC_pi]
| Surface | Guide | Engine |
|---|---|---|
| Windward wall (0–15) | 11.4 / 21.9 | ✅ |
| Leeward wall | −17.8 / −7.2 | ✅ |
| Side wall | −22.8 / −12.2 | ✅ |
| Windward roof (−0.36) | −14.3 / −3.7 | ✅ |
| Windward roof (+0.14) | −1.8 / 8.8 | ✅ |
| Leeward roof (−0.57) | −19.5 / −8.9 | ✅ |

### Net design pressures, parallel to ridge (Table G3-7), psf
| Surface | Guide | Engine |
|---|---|---|
| Leeward wall (−0.45) | −16.5 / −5.9 | ✅ |
| Roof 0→h (−0.9) | −27.8 / −17.2 | ✅ |
| Roof h→2h (−0.5) | −17.8 / −7.2 | ✅ |
| Roof >2h (−0.3) | −12.8 / −2.2 | ✅ |

The integrated `compute_loads` path (member line load ÷ tributary) also returns
the windward-wall **11.4 psf**.

---

# Example #2 — low-slope gable (θ = 5°), independent hand calc

Exercises the **zoned windward-roof, normal-to-ridge** path (θ<10°, ASCE 7-22
Fig 27.3-1 flat-roof treatment) — the code path G3-1 does **not** reach (its 18.4°
roof is uniform normal to ridge). Test: [`../tests/test_validation_lowslope.py`](../tests/test_validation_lowslope.py)
(7 tests, tol **0.3 psf**).

## Building (hand calc)
- Span (⊥ ridge) **L = 100 ft**, length 150 ft, **eave 30 ft**, **θ = 5°**
- Frames @ 25 ft o.c. (7); **Exposure C**, **V = 120 mph**, K_zt=K_e=1, **K_d=G=0.85**
- **Enclosed** → GC_pi = ±0.18 · Units **US**

## Derived (ASCE 7-22 Ch 26-27 + Fig 27.3-1)
- ridge = 30 + 50·tan5° = **34.37 ft**; **h = 32.19 ft**; **h/L = 0.322** (≤0.5)
- K_z(h) = 2.41·(32.19/2460)^(2/9.8) = 0.995 → **q_h = 36.67 psf**
- K_z(15) = 0.851 → **q_z15 = 31.37 psf**
- roof zones (h/L≤0.5): 0→h **C_p=−0.9**, h→2h **−0.5**, >2h **−0.3**

### Net design pressures, normal to ridge (psf) — [+GC_pi, −GC_pi]
| Surface | C_p | Hand | Engine |
|---|---|---|---|
| Windward wall (0–30, q_z15) | 0.80 | 12.5 / 23.7 | ✅ |
| Leeward wall | −0.50 | −18.9 / −7.6 | ✅ |
| Side wall | −0.70 | −24.2 / −12.9 | ✅ |
| Roof zone | −0.90 | −29.5 / −18.2 | ✅ |
| Roof zone | −0.50 | −18.9 / −7.6 | ✅ |
| Roof zone | −0.30 | −13.6 / −2.3 | ✅ |

The integrated `compute_loads("+X")` returns the windward roof as **two zones**
(−0.9 then −0.5 across x=0→ridge) and the leeward roof as −0.5 then −0.3, matching
the hand pressures — confirming the zoned-normal path end to end.

## Live oriented-matrix campaign — 2026-06-10

The runbook campaign (`prototypes/run_oriented.py`, **full matrix**) was executed
live against MIDAS GEN NX 2026 (kr relay):

- **Result: 104/104 PASS** (13 scenarios × 8 placement schemes; zero
  FAIL/ERROR/SUSPICIOUS lines in the log). Runtime ≈ 39 min (~20–25 s/run).
- Each run verified all four PASS conditions: geometry round-trip
  (±0.01 m / 0.1°), zero warnings, exactly **58 cases** (Dead, Roof Live,
  8 directional wind, 48 MWFRS C1–C4), and the **world-frame net (GX, GY)** of
  `Wind +X +GCpi` equal to the source local vector rotated by the scheme angle
  (to 0.1%).
- Log: [`runs/oriented_20260610_2018.log`](runs/oriented_20260610_2018.log).

This closes the live verification of pull recovery + load orientation across
arbitrary placement (translate, podium level, rotations incl. 33.2°, combined).

## Status & remaining
- ✅ **G3-1** — numeric agreement with the official ASCE 7-22 Guide example.
- ✅ **Low-slope θ<10°** — independent hand calc; zoned-windward-roof-normal path.
- ✅ **Oriented matrix (live)** — **104/104 PASS**, 2026-06-10 (see above).
- ⬜ Independent licensed-engineer review (recommended before production use).
- ⬜ (Optional) a monoslope worked example to validate §11d Cp reuse numerically.

> Sources: ASCE/SEI 7-22 Wind Loads Guide, Example G3-1 (user-provided); Example #2
> is an independent hand calc from ASCE 7-22 Ch 26-27. Transcribed/derived under the
> user's licensed copy.
