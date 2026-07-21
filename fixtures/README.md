# Marker-DXF import fixtures & regression harness

Hand-authored, minimal ASCII DXFs exercising the BLOCK/INSERT resolver
(`resolveBlockInstances` in `index.html`) and the marker-DXF import
pipeline, plus golden-file expectations checked by **`../tests.html`**
(Phase 3 of the DXF interoperability roadmap). Each fixture's expected
output is known analytically.

## Running the harness

Serve the app folder over HTTP and open `../tests.html`. It boots the
real app in an iframe at `index.html?test=1` (a test flag that skips
localStorage restore and never writes back, so your saved workspace is
never touched), then for every fixture in `manifest.json` runs the
declared modes and deep-compares the result against the golden in
`expected/`. Three built-in invariants also run: the Phase-1 0.00 cm
Seamline round trip, AAMA-package stability, and unit-scale + Y-flip.
Green banner = all pass. `window.__results` holds the summary for a
future headless runner.

Load a fixture manually via `parseAamaDxf(text)` then
`window.SeamlineMarkerDxf.resolveBlocks(dxf, '1')`, or the full
`window.SeamlineMarkerDxf.inspect(text, filename)` /
`.build(info, width, unitScale, flipY)` pipeline, or the
"Import marker DXF…" dialog.

## Manifest & goldens

- `manifest.json` — one entry per fixture: `{file, modes, units, flipY,
  width, notes}`. `modes` ⊆ `resolve` (raw `resolveBlocks` output),
  `inspect` (the counts/reasons the import summary is built from),
  `build` (the placed marker, uid-free), `throws` (asserts `inspect`
  throws with the expected message).
- `expected/<fixture>.<mode>.json` — the golden. `resolve`/`inspect`/
  `build` goldens are derived from the **data** the UI warnings are
  computed from (never the rendered HTML), so a weakened or silenced
  warning fails the diff. Coordinates compare at ±0.01 cm (the app's own
  2-dp rounding). uids never appear — cut order is stored as piece
  indices.

## Adding a real production vendor DXF

1. Drop `vendor-<system>-<desc>.dxf` into this folder. **Sanitize any
   customer-identifying label text first** — it gets committed.
2. Add one `manifest.json` entry; choose `units`/`flipY`/`width` while
   reviewing (the harness shows a NO-GOLDEN row and an SVG geometry
   preview so you can eyeball it).
3. Click that row's **Record** (or **Record all goldens**) to download
   the normalized JSON, review it, and move it into `expected/`.
4. Re-run → the row goes green. Commit the `.dxf`, the manifest entry,
   and the golden(s) together.

All fixtures place a 10×10 unit square block (`PIECE_A`, local origin
`(0,0)`–`(10,10)`) unless noted. Coordinates below are in DXF file units,
pre unit-scale/Y-flip (those are applied later by `buildMarkerDxfMarker`).

| Fixture | Exercises | Expected `resolveBlocks` result |
|---|---|---|
| `fixture-01-simple-insert.dxf` | Plain INSERT, translate only | 1 ring: `(5,5)-(15,5)-(15,15)-(5,15)` |
| `fixture-02-rotated-insert.dxf` | `rot=90` | 1 ring: `(0,0)-(0,10)-(-10,10)-(-10,0)` — confirms `(x,y)→(-y,x)` |
| `fixture-03-mirrored-insert.dxf` | `sx=-1` (mirror, no dedicated flag) | 1 ring: `(0,0)-(-10,0)-(-10,10)-(0,10)` — reversed winding, no special-casing needed |
| `fixture-04-nested-insert.dxf` | Container block (`ASSEMBLY`, no own contour) holding a nested INSERT of `PIECE_A` | 1 ring, composed transform `(100,100)∘(2,3)` → `(102,103)-(112,103)-(112,113)-(102,113)`. **No `no-contour` warning** — a block with nested INSERTs and no own boundary is normal scaffolding. |
| `fixture-05-missing-block.dxf` | INSERT referencing an undefined block (`GHOST`) | `rings=[]`, `unresolved=[{block:'GHOST',reason:'missing-block'}]`. Note: run through `resolveBlocks` directly — the full `inspect()` pipeline throws "No closed contour found" for this file alone, same as any file with zero placeable geometry (correct, matches Phase 1's existing contract). |
| `fixture-06-circular-reference.dxf` | `LOOP_A` inserts `LOOP_B` inserts `LOOP_A` | 2 rings kept (the legitimate `LOOP_A` at `(100,100)-(104,100)-(104,104)-(100,104)` and `LOOP_B` at `(110,100)-(112,100)-(112,102)-(110,102)`), plus `unresolved=[{block:'LOOP_A',reason:'circular-reference'}]`. Confirms cycle detection doesn't discard already-resolved siblings/ancestors. |
| `fixture-07-mixed-flat-and-block.dxf` | Top-level flat contour (Phase 1 style) + BLOCK/INSERT contour in the same file | 2 total placed pieces via the full `inspect()` pipeline: one from `info.contours` (the 30×15 rectangle at `(200,200)`), one from `info.blockRings` (the `PIECE_A` square at `(5,5)`). Confirms both import paths coexist. |
| `fixture-08-array-insert.dxf` | INSERT with `cols=2` | 1 ring (the base instance at `(5,5)`) plus `arrayInserts=[{block:'PIECE_A'}]`. The repeat grid is reported, not silently expanded — kept separate from `unresolved` because a piece *was* placed. |
| `fixture-09-max-depth-chain.dxf` | 9-deep nested chain `CH0→…→CH8` | 8 rings (`CH0`..`CH7`, 2×2 squares marching right by 10) plus `unresolved=[{block:'CH8',reason:'max-depth'}]`. Locks the depth cap (`MDXF_MAX_DEPTH=8`) and confirms it bounds work without discarding the levels already resolved. |
| `fixture-10-no-contour-block.dxf` | Block `NC` holding only a `LINE`, no nested INSERTs | `rings=[]`, `unresolved=[{block:'NC',reason:'no-contour'}]`. The distinguishing case from fixture-04: no boundary **and** no nested inserts is a genuine warning. |
| `fixture-11-nonuniform-scale.dxf` | `sx=2, sy=1` | 1 ring `(0,0)-(20,0)-(20,10)-(0,10)`; the `build` golden confirms def area `200`. |
| `fixture-12-bulge-in-rotated-block.dxf` | `bulge=0.5` edge inside a block inserted at `rot=90` | Ring tessellated **in the block-local frame first, then rotated** — the arc stays circular. Golden pins the exact tessellated point list (regression lock on flatten-then-transform). |
| `fixture-13-attrib-metadata.dxf` | INSERT with `group 66=1` and `PIECE NAME`/`SIZE`/`BUNDLE` ATTRIBs | `build` golden: def name `FRONT`, size `M`, bundle `3` — all sourced from ATTRIB rather than a layer-15 text label. |
| `fixture-14-mm-flat-contour.dxf` | Flat mm contour built at `unitScale=0.1`, `flipY=true`, `width=20` | `build` golden: 10×5 cm piece (area `50`) at min `(0,15)` after the millimetre conversion and Y flip. |

## Regression guarantee

None of these fixtures should ever affect Phase 1 behavior — a Phase 1
file (Seamline's own `exportDXF` output) has no `BLOCKS` section and no
`INSERT` entities, so `resolveBlockInstances` always returns
`{rings:[], unresolved:[], arrayInserts:[], blockDefCount:0}` for those
files: a true no-op.
