# Seamline Marker

Interactive marker-making module of the **Seamline** apparel software bundle.
One self-contained file — **double-click `SeamlineMarker.html`** to run it in any browser. No install, no server; work is autosaved in the browser between sessions.

## Workflow (modeled on the classic marker pipeline)

1. **New Order** — an Easy Order-style 5-step form (Models → Sizes → Pieces → Fabric → Finish): add **one or more models** (T-Shirt, Shirt, Trousers, Bra, Panty) as order lines, set size quantities per model, untick pieces to exclude them, **set each piece's lay direction** (two-way by default, overridable to four-way or free per piece), choose fabric width / cutter buffer / lay rule, review the summary with an estimated marker length, then **Create** or **Create + Auto-Nest**. All models nest together in one combined marker; pieces carry a model tag (TSH·, SHT·, TRS·) when the order is multi-model.
2. **Place** — click a tray piece then click the fabric (green ghost = valid, red = blocked), or press **⚡ Auto-Nest** to pack the whole marker automatically (effort slider trades speed for tightness).
3. **Refine** — drag pieces (overlaps bounce back), rotate/flip under lay-rule constraints, **Butt** slides a piece until it touches its neighbour, **Compact** tightens the whole marker. Length, **utilization %** and piece counts update live; status chip = MADE / PARTIAL / UNMADE.
4. **Output** — **Save** the marker as JSON (re-open later with **Open**), export the plot as **SVG** (1 unit = 1 cm, with title block) or **PNG**.
5. **Cut Plan** (Easy Plan analog) — for bulk production: enter demand per **colour × size** plus ply limits (min/max ply, max garments per marker), hit **Calculate markers**, and the demand is split into a marker set with spreads (colour × plies) — proportional demand collapses into one shared ratio marker; residuals are cleared exactly by per-colour cleanup markers, with below-min-ply spreads flagged. **Auto-nest all markers** draws every layout in the plan and replaces the length estimates with actual nested lengths and utilizations. **Mini-markers PDF** downloads the whole plan as a dependency-free PDF — a summary page (demand, settings, marker table) plus one page per marker with the nested layout drawing, ratio, spreads, plies, garments, length, utilization and fabric usage. Open any plan row as a marker order, or export the cut tickets as CSV.

## Rules engine

- Pieces can never overlap; an optional buffer (0.3 / 0.5 / 1 cm) keeps a minimum cutter gap between pieces.
- Every piece carries a lay direction — body panels default to two-way (0°/180°), small parts four-way, bindings free — set per piece in the order form, with a global lay rule (strict / 90° ok / free) that can further relax (never tighten) all pieces at once for non-directional fabric.
- Paired pieces (shirt fronts, trouser legs) are cut mirrored automatically.
- **Size scales** are per model, so garments on different sizing systems can share one marker:
  - `alpha` — XS–XXL (T-Shirt, Shirt, Trousers, Panty). Length grades at half the girth rate.
  - `bra` — **32A–42D** (24 sizes: bands 32–42 × cups A–D). Two-dimensional grading: each piece declares what drives it, so band size grows the wings, under band and straps, while the cup letter grows the cup panels — a 36D has the same 68 cm band as a 36B but a larger cup. Size chips are coloured by hue = cup, shade = band.
- Adding a model is data-only: give it a piece list and (optionally) a `scale`; the order form, cut plan, marker labels and PDF pick it up automatically.

## Keyboard

`R`/`Shift-R` rotate · `F`/`Shift-F` flip · arrows nudge 1 cm (`Shift` = 1 mm) · `L`/`U` butt-slide · `Del` return to tray · `Esc` cancel · `Ctrl-Z`/`Ctrl-Y` undo/redo · `+`/`−`/`0` zoom · drag background to pan · `N` new order · `?` help

## Reference

Feature set derived from the AccuMark Easy Marking reference charts in the parent folder (`EasyMarker_Charts.md` and the `EM_*.mmd` diagrams): order entry → order process → interactive/automatic placement → marker status → plot/cut output.
