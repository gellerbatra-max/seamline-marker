# Marker-DXF import fixtures

Hand-authored, minimal ASCII DXFs exercising the BLOCK/INSERT resolver
(`resolveBlockInstances` in `index.html`) added in Phase 2 of the DXF
interoperability roadmap. Each fixture's expected output is known
analytically — use these as Phase 3 regression assertions.

Load a fixture as text and pass it through `parseAamaDxf(text)`, then
`window.SeamlineMarkerDxf.resolveBlocks(dxf, '1')`, or run it through the
full `window.SeamlineMarkerDxf.inspect(text, filename)` pipeline via the
"Import marker DXF…" dialog.

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

## Regression guarantee

None of these fixtures should ever affect Phase 1 behavior — a Phase 1
file (Seamline's own `exportDXF` output) has no `BLOCKS` section and no
`INSERT` entities, so `resolveBlockInstances` always returns
`{rings:[], unresolved:[], arrayInserts:[], blockDefCount:0}` for those
files: a true no-op.
