# Seamline Marker

Interactive marker-making module of the **Seamline** apparel software bundle.
One self-contained file — **double-click `SeamlineMarker.html`** to run it in any browser. No install, no server; work is autosaved in the browser between sessions.

## Workflow (modeled on the classic marker pipeline)

1. **New Order** — an Easy Order-style 5-step form (Models → Sizes → Pieces → Fabric → Finish): add **one or more models** (T-Shirt, Shirt, Trousers) as order lines, set size quantities per model, untick pieces to exclude them, choose fabric width / cutter buffer / lay rule, review the summary with an estimated marker length, then **Create** or **Create + Auto-Nest**. All models nest together in one combined marker; pieces carry a model tag (TSH·, SHT·, TRS·) when the order is multi-model.
2. **Place** — click a tray piece then click the fabric (green ghost = valid, red = blocked), or press **⚡ Auto-Nest** to pack the whole marker automatically (effort slider trades speed for tightness).
3. **Refine** — drag pieces (overlaps bounce back), rotate/flip under lay-rule constraints, **Butt** slides a piece until it touches its neighbour, **Compact** tightens the whole marker. Length, **utilization %** and piece counts update live; status chip = MADE / PARTIAL / UNMADE.
4. **Output** — **Save** the marker as JSON (re-open later with **Open**), export the plot as **SVG** (1 unit = 1 cm, with title block) or **PNG**.

## Rules engine

- Pieces can never overlap; an optional buffer (0.3 / 0.5 / 1 cm) keeps a minimum cutter gap between pieces.
- Every piece carries a lay rule — body panels are two-way (0°/180°), small parts four-way, bindings free — with a global override (strict / 90° ok / free) for non-directional fabric.
- Paired pieces (shirt fronts, trouser legs) are cut mirrored automatically.
- Grading: XS–XXL scale factors, length grades at half the girth rate.

## Keyboard

`R`/`Shift-R` rotate · `F`/`Shift-F` flip · arrows nudge 1 cm (`Shift` = 1 mm) · `L`/`U` butt-slide · `Del` return to tray · `Esc` cancel · `Ctrl-Z`/`Ctrl-Y` undo/redo · `+`/`−`/`0` zoom · drag background to pan · `N` new order · `?` help

## Reference

Feature set derived from the AccuMark Easy Marking reference charts in the parent folder (`EasyMarker_Charts.md` and the `EM_*.mmd` diagrams): order entry → order process → interactive/automatic placement → marker status → plot/cut output.
