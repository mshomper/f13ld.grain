# F13LD.grain — UI refresh log

| Stage | Date | Branch | Commit | Checks | Open decisions |
|---|---|---|---|---|---|
| 1. Audit | 2026-11-25 | `ui-refresh` | none yet | 0 page errors (390/700/1024/1440); no horizontal scroll at 390/700/1024/1440; 0 compiles per slider drag (isoLevel, thickness, kappa); 65 controls mapped; 30 help `i` | accent (violet vs aqua); bake resolution once Low/Med/High/Ultra goes; smoothing (remove or make it work); seed into Field; Direction section for RD; auto-rotate default; `hu_lock_scale` export key; auto-simulate after RD import; page title |
| 2. Layout shell | 2026-11-25 | `ui-refresh` | `3f76ae1` | 0 page errors (390/700/1024/1440, all sections and field types, help open); 0 horizontal scroll and 0 text under 11 px and 0 targets under 44 px across 4 widths x 11 states (hit areas on `::after` count); export JSON identical to `main` for 8/8 states (spinodoid x4, GRF, HU, RD Gray-Scott, RD Brusselator); homogenization block identical to `main`, cleared after a design change; help `i` 30 -> 29 (smoothing card removed), 35 cards; help "show" lands on its control 70/70 (35 cards x desktop/phone); 25/25 sliders carry `data-w-slider`; 33 screenshots `docs/ui-audit/stage2-*.png` | quality switch still native (stage 4); controls still native (stage 3); desktop section rail and phone tab bar are plain fallback buttons until the liquid widgets mount; initial camera distance 16 -> 19 so the cube is not clipped in the larger viewer; `#exportBtn` stays in Analyze until the Recipe menu (stage 3); stiffness glyph loop still runs forever (stage 4) |

## Decisions (Matt, 2026-11-25)

Shared, all wave 1 tools:
- **Quality:** automatic, as in TPMS. Remove the manual quality control (buttons or slider); its help (i) goes with it.
- **Stale analysis:** clear homogenization / stiffness results when the design changes, and on import. Exports carry results only for the current design.
- **Missing help cards:** add them; the (i) count rises by that number (record the new count).
- **Field sliders that re-bake:** show a low-res preview bake while dragging, full bake on release. Keep it only if it measures faster to first change than today and does not slow the full bake.
- **Accent:** derived from the tool colour in the F13LD brand guidelines v2.0 (hue kept, lifted to L 0.80 for the dark UI). Final value pending Matt.

This tool:
- **Accent (proposed):** stone `oklch(0.80 0.028 93)` ≈ `#c4beaa`, from brand stroke `#3D3A30` (warm grey). Replaces the violet in the audit.
- **Bake resolution:** fixed 128³ (today's High). Ultra 256³ is 8× the voxels and falls back to the main thread without the worker. Preview bake 48³ while a field slider moves. RD keeps its own 48³ grid.

- Remove the `smoothing` slider; keep its export key at the default value. RNG seed moves into Field. Direction section disabled for reaction-diffusion with a one-line note. Auto-rotate off at start. Add `hu_lock_scale` export key (check Mesh/Lab ignore unknown keys first). Run the RD simulation automatically after importing an RD recipe. Page title "F13LD.grain"; fix the stale "Section 2D" hint.

Stage 1 approved by Matt 2026-11-25 ("use your picks").
