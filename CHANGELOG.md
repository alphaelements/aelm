# Aelm Changelog

The canonical release note for the **Aelm toolchain** as a whole — the
Cargo workspace, the `aelm` CLI, the VSCode extension, and the
`@alphaelements/aelm-renderer` npm package all bump in lockstep, and
this file is the cross-artifact record of what changed in each
release.

Per-store changelogs live next to each artifact's manifest:

- VSCode Marketplace surface: [`vscode-extension/CHANGELOG.md`](./vscode-extension/CHANGELOG.md)
- npm registry surface: [`web/renderer/CHANGELOG.md`](./web/renderer/CHANGELOG.md)

Format: [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/).
Versioning: [SemVer 2.0.0](https://semver.org/spec/v2.0.0.html).
Authoring: [`knope prepare-release`](./knope.toml) consumes the
Conventional Commits between releases and produces the Unreleased
section below; reviewers polish the wording before the release tag is
cut via `scripts/release.sh`.

## [Unreleased]

## [0.5.0] - 2026-05-20

### Added

- **Interactive `calcs {}` block.** A new annotation kind that
  embeds editable variables and live formula results directly on
  the schematic canvas. Variables (`R: var 10k type: Ohm`) render as
  bordered text-input boxes; formulas (`fc: formula "1/(2*pi*R*C)"
  display: "$f_c = {fc}$ Hz"`) compute against the current variable
  environment and re-typeset their KaTeX `$...$` math fences through
  the same glyph pipeline as notes. Editing a variable in the
  WebView updates the source `var` value and re-evaluates dependent
  formulas without re-running parse / layout / routing. Chained
  formulas resolve through dependency-ordered evaluation; cycles,
  duplicate ids, unknown identifiers, and malformed expressions
  surface as `E_CALC_001`–`E_CALC_006` diagnostics. Variable and
  formula entries can be dragged, locked at any `lock:` level,
  hidden via `visible: false`, and deleted from the canvas;
  reverse-sync writes preserve indentation and adjacent hints.
  Example: `examples/calc-lpf.aelm` (RC low-pass cut-off frequency
  + ω calculator).
- **Circuit analysis commands.** Five new read-only `aelm analyze`
  subcommands surface project structure for tooling and agents.
  `aelm analyze summary` reports per-module instance / connection / net
  counts, the parts used (with category and quantity), a DRC error /
  warning summary, and a complexity estimate. `aelm analyze netlist`
  extracts the net connectivity graph — each net's name, electrical
  type, and member pins — derived from the union-find net map.
  `aelm analyze bom` generates a bill of materials grouped by part and
  value, with reference designators, footprint, and quantity.
  `aelm analyze connectivity` reports isolated instances and per-net
  fan-out. `aelm analyze extract --instances A,B,C` emits a standalone
  module containing only the requested instances and their mutual
  connections. All accept a file path or `--stdin` and honor the unified
  `--json` envelope.
- **Part, symbol, and example discovery commands.** Three new CLI
  command groups let tools and humans browse the library surface
  without authoring a circuit. `aelm parts list|info|search` enumerates
  parts from the stdlib and any user libraries (`-L <dir>` or
  `--from <file>.alib`), with fuzzy search over name, description,
  categories, and MPN, and an optional inline symbol SVG via
  `--render-symbol`. `aelm symbols list|info|render` enumerates the
  built-in symbol templates plus user-defined `symbol {}` blocks,
  grouped by category, and renders any template to a standalone SVG
  (`--params left=4,right=4` for parametric symbols). `aelm examples
  list|get` browses the bundled example circuits, returning source and
  an optional rendered SVG. All three honor the unified `--json`
  envelope.
- **MCP-friendly CLI: structured JSON, stdin piping, dry-run.** Every
  command can now emit a unified `{ success, data, diagnostics }` JSON
  envelope via the global `--json` flag, and read source from a pipe via
  `--stdin` (`parse`, `check`, `fmt`, `layout`) — eliminating temp files
  for programmatic consumers. `fmt` gains `--output -` for stdout-only
  formatting; every `apply-*` command gains `--dry-run` (write the
  modified source to stdout, leave the file untouched) and `--json`
  (modified source plus a per-line change diff). `lib validate --json`
  reports lint findings as structured data. A new `aelm pipeline`
  compound command runs `parse`, `check`, and `render` stages in one
  process (`--stages parse,check,render`), sharing the parse result so
  it is markedly faster than three separate invocations; SVG is embedded
  as text and PNG as base64 in the JSON output. Diagnostic spans in JSON
  output use `{ line, column, length }`. Existing text/in-place behavior
  is unchanged when the new flags are absent.
- **MCP server (`aelm-mcp`).** A standalone Model Context Protocol
  server (`aelm-mcp` submodule) exposes the full Aelm toolchain to
  AI agents over stdio. 34 tools (parse, validate, format, render
  SVG/PNG, parts/symbols/examples catalog, project analysis, dry-run
  edit, scaffold generation, structural diff, batch render, style
  preview, placement suggestions, calc evaluation, pattern search,
  circuit similarity), 7 workflow prompts (design, review, debug DRC,
  part selection, tutorial, explanation, interactive design loop), and
  6 embedded reference-doc resources. Built on `rmcp 1.7`, invokes
  the `aelm` CLI via subprocess. Version lockstep with the parent
  workspace via `scripts/release.sh`.
- **Signal connection labels.** Five new builtin symbols
  (`signal_label_input`, `signal_label_output`, `signal_label_bidir`,
  `signal_label_tristate`, `signal_label_passive`) for annotating signal
  direction on nets. Nets declared as `input`/`output`/`bidirectional`/
  `passive` now default to the corresponding signal label shape instead
  of `power_flag`. New `stdlib/parts/signal.alib` provides matching
  stdlib parts.
- **Calc variable label position.** Calc `var` entries accept a new
  `label_pos: above | below | left | right` inline hint (and the
  matching `label_pos:` declaration inside the stylesheet `calc_var`
  selector) to relocate the variable name + unit label relative to the
  input box. Defaults to `above` for backwards compatibility. Inline
  hint wins over the stylesheet value.
- **Calc-driven plot expressions.** Trace expressions inside
  `plots {}` now resolve any identifier defined in the same module's
  `calcs {}` block. Editing a variable in the WebView reflows every
  dependent plot without re-running parse / layout. The sweep variable
  `x` always wins over a same-named calc identifier.
- **Logarithmic plot axes.** Any axis accepts `log: true`. Sampling,
  coordinate mapping, and tick generation all switch to log10 — Bode
  magnitude/phase, impedance plots, thermal response, dynamic range
  and other multi-decade engineering plots render correctly without
  any data-side transformation. Works on x / y / y2 independently;
  data points with non-positive coordinates on a log axis are skipped.
- **Minor (sub) grid lines.** Plot axes accept `sub: true` (log axes
  draw 2×–9× lines per decade) or `sub: <N>` (linear axes divide each
  major interval into N parts), rendered as faint lines beneath the
  major grid.
- **Expression-driven axis range and scale.** `range:` and `scale:`
  accept expressions referencing calc identifiers
  (e.g. `range: fc / 100 .. fc * 100`), resolved against the live calc
  environment at plot-evaluate time, so a plot's window tracks its
  component values automatically. Numeric literals continue to work
  unchanged.
- **Calc box and formula styling.** Calc `var` boxes and `formula`
  cards honour `style { font_size; background; font_weight }`. Boxes
  scale with the font size and grow with the value width; variable
  name labels render bold by default (opt out with
  `font_weight: normal`).

### Changed

- **Calc rendering is theme-aware.** Calc variable boxes and formula
  cards now pick a light or dark palette directly from the host theme,
  so labels and values stay legible (near-white) against a dark canvas
  instead of relying on a post-render colour remap. Plot major/minor
  grid contrast in dark mode was also tuned up.

### Fixed

- **Net-tap drag duplicates `place:` clauses.** Moving a net-tap that
  had a `{ label_offset: ... } place: ...` shape used to append a
  second `place: (gx, gy)` clause every drag because the reverse-sync
  helper stopped at the first `{`. The post-body `place:` is now
  located and updated in place.

- **Calc block drag uses world coordinates.** Earlier drag handlers
  passed `(0, 0)` as the source position, so the first drop after a
  reload could snap to a stale offset. Drags now hand the current
  centre and snapped target centre to the reverse-sync mover, so
  the rewritten `place:` matches the on-screen position exactly.

## [0.4.2] - 2026-05-19

### Added

- **Parametric curves, patterns, and mirror figures.** New
  `parametric`, `pattern`, and `mirror` figure kinds extend the
  `figures {}` block. Parametric curves accept `x:` / `y:` expressions
  in `t` (with `sin`, `cos`, `pow`, `atan2`, `min`, `max`, `abs`,
  `sqrt`, `exp`, `log`) sampled over a `t: (start, end, step)` range
  and rendered as polylines or dot clouds (`connect: line | none`).
  Patterns replicate a source figure on a `rotate:` or `axis_replicate:`
  (grid) axis. Mirrors emit a reflected copy across `axis: x` /
  `axis: y` or an arbitrary `axis_line: from (x1,y1) to (x2,y2)`. Source
  figure `place:` offsets propagate through mirror expansion so
  reflected copies follow the source when it moves. Expanded copies
  (`<id>#<n>`) advertise `Movable`, `Lockable`, `Highlightable`, and
  `ToggleVisibility` capabilities; dragging shifts the mirror metadata's
  `place:` so the reflection tracks the new position. Hovering an
  expanded mirror copy draws the mirror axis as a dashed guide line
  (vertical for `axis: x`, horizontal for `axis: y`).
- **Half-grid snapping for figure placement.** Figure `place:` and
  WebView drag output now snap to 1.27 mm half-grid steps, written
  back as 0.5-step full-grid units (e.g. `place: (2.5, 1)`). Figure
  grammar coordinates accept floats so authors can hand-write
  half-grid positions.
- **`aelm.zoomToFit` command** centres and scales the WebView so the
  whole circuit fits with a 10% margin. Available from the command
  palette, the editor title-bar (screen-full icon), the canvas
  overlay button, and the `Ctrl+0` / `Cmd+0` keybinding. Works across
  `circuit`, `block_diagram`, and `flow_chart` modules. Backed by a
  new `compute_world_bounds` helper in `aelm-render` that aggregates
  components, routed wires, notes, figures, and plots.
- **`aelm.resetZoom` command** restores the viewport to the default
  centre (0, 0) at 15 px/mm — the same state a freshly-opened editor
  starts in. Available from the command palette, the editor title-bar
  (zoom-out icon), the canvas overlay button, and the
  `Ctrl+Shift+0` / `Cmd+Shift+0` keybinding.
- **`get_circuit_bounds` WASM API** returns the world-space bbox of a
  laid-out module, consumed by the VSCode `zoomToFit` command and
  available to any other host (npm renderer, screenshot scripts).
- **PNG export via Canvas API.** `AELM: Export as PNG` now rasterizes
  the SVG through the WebView Canvas at 2× resolution instead of
  showing "not available". Default file name is derived from the
  active `.aelm` document.
- **`max_pixels` export option.** `ExportConfig` accepts a
  `max_pixels` cap (default 8910 = A2 landscape at 15 px/mm) to
  prevent oversized SVG/PNG output on large diagrams.
- **Net-tap per-tap symbol override.** Individual taps can now pick a
  different flag glyph from the net default via
  `G2: net GND { symbol: gnd_triangle }`, enabling mixed ground symbols
  (e.g. signal vs chassis ground) on the same electrical net.
- **`label_only: true` nets.** Nets declared with `label_only: true`
  render taps as text-only labels with no flag glyph, useful for signal
  nets like SDA/SCL that join distant pins by label only.
- **Auto-migration of legacy flag instances.** `GND1: GND` / `VCC1: PWR`
  style stdlib flag instances are automatically migrated into net-taps at
  render time, so all instances sharing an inferred net name display a
  unified label instead of per-instance IDs.
- **Bare net-ref nearest-tap routing.** `-> GND` in connections now
  resolves to the nearest physical tap by Manhattan distance, producing
  correct point-to-point routing instead of ambiguous net-wide connections.
- **Net-tap label position offset.** New body entry
  `label_offset: (dx, dy)` shifts the rendered label by `dx` / `dy` grid
  units from the default anchor, allowing authors to nudge labels off the
  wire they connect to.
- **Three-direction label text rotation for label-only taps.** When a
  label-only tap is rotated 90° / 180° / 270°, the text itself rotates to
  follow the wire while never appearing upside-down (0° / -90° / +90°).
- **Interactive net-tap label drag.** The WebView emits a separate
  draggable handle on the label so users can reposition it
  independently of the flag glyph. Drags snap to half-pitch (0.5 grid)
  and the result is written back to the source as `label_offset:`. The
  R key on the label rotates the underlying tap.

### Fixed

- **Figure bbox offset pitch matches render.** Interactive hit-test
  boxes on figures with a `place:` offset now line up with the drawn
  geometry — they previously used a different pitch from the renderer,
  making offset figures hard to click.
- **Mirror axis reference line direction.** Hovering an expanded mirror
  copy now draws a *vertical* dashed line for `axis: x` (the line the
  reflection mirrors across) and a *horizontal* line for `axis: y`. The
  previous build drew them swapped.
- **Figure-grammar accepts float coordinates** (e.g. `place: (2.5, 1)`)
  so half-grid placements can be hand-written without the parser
  rejecting the source.
- **CSV plot traces now render in the WebView.** Files with
  `plots { source: "..." }` but no `use` statements triggered an
  early-return in `resolveImports`, causing the pipeline to skip
  `parse_with_imports_v2` and lose the CSV file map. The axis frame
  rendered but the waveform polyline was missing.
- **SVG / PNG exports include CSV plot traces.** The `exportSvg`
  path used `parse()` instead of `parse_with_imports_v2()`, so CSV
  data never reached the renderer. Imports are now resolved before
  export.
- **PNG export no longer times out on large diagrams.** Canvas
  dimensions are capped at 8192 px per axis (scaled proportionally)
  and the rasterization timeout is increased from 10 s to 30 s.

### Changed

- **Examples and tips cleaned up.** Removed unnecessary `use` and
  dummy `Resistor` instances from plot-only / figure-only files.
  Aligned `aelm-version` headers and library metadata to 0.4.2.
  Added gallery files for parametric curves and plot samples.
- **Marketplace preview flag removed.** The extension is no longer
  marked as "Preview" on the VS Code Marketplace.

## [0.4.1] - 2026-05-15

### Added

- **`flow_node` / `block` label rendering inside the node body.** When
  `flow_node(..., label: "X")` or `block(..., label: "X")` is declared,
  the renderer now draws `"X"` inside the shape instead of the instance
  identifier. ASCII and multibyte (Japanese / CJK) labels survive
  verbatim through the parser → IR → render → SVG/Canvas pipeline. The
  identifier is still used for `flow { A -> B }` references and DRC
  messages — only the *drawn* text changes. Long labels can still
  overflow the fixed bbox; auto-fit and multi-line wrapping are tracked
  as separate follow-ups.
- **`border_radius` for `figure_rect` + `figure { }` style cascade.**
  `figure_rect` now accepts an inline `style { border_radius: <value>; }` (in mm
  or as a bare number) that rounds all four corners via SVG `rx`/`ry` and a
  manual arc path on Canvas. An external `.astyle` file can set defaults for every
  figure in the diagram using the new `figure { }` pseudo-element:
  ```astyle
  figure {
      stroke_color: #004488;
      border_radius: 1.5mm;
  }
  ```
  Inline `style {}` overrides the sheet, backward-compatible (absent/zero = sharp
  corners, no rx/ry emitted).
- **`polygon`/`polyline` CSV row-delimiter shorthand.** `points:`
  now accepts bare `x, y` pairs separated by newlines (no parentheses
  required), mirroring the `samples:` shorthand in `plots {}`. Both
  forms are interchangeable and can be mixed in the same list.
- **Figure polygons + vertex handles.** Three new primitives —
  `triangle place: (x,y) size: (w,h)`, `polygon place: ... points:
  [(dx,dy), ...]`, and `polyline from: ... points: [(dx,dy), ...]` —
  join the seven Phase 2 shapes in the `figures {}` block. Polygons
  and polylines store vertices as offsets from the anchor, so a
  drag-to-move shifts the anchor only and the vertex list stays
  byte-identical. New `apply_figure_vertex_move(source, figure_id,
  vertex_index, new_x, new_y)` WASM API rewrites a single vertex
  in `points: [...]`; the WebView surfaces one drag handle per vertex
  (`InteractiveKind::FigureVertex`, ids `figure:<fid>:v<idx>`) and
  draws them on top of every overlay so they pick up the click first.
  `FigureStyle` gains `stroke_dash`, `stroke_cap`, and `stroke_join`;
  the renderer now builds every figure stroke through one
  `figure_stroke_style()` helper so a dashed rect / dotted circle /
  rounded-cap arc all work the same way. New diagnostic `E_FIG_005`
  rejects polygons with fewer than two points and polylines with no
  points.
- **Figure interactivity.** The `figures {}` block now lights up the
  same WebView gestures notes already support: click selection with
  source navigation, drag-to-reposition (snapped to grid; rewrites
  `place:` / `from:` / `to:` / `center:` together so a `line` figure
  shifts both endpoints in one call), `R` for 90° rotation, `L` for
  lock-level toggle, `V` for visibility toggle, `Del` for deletion, and
  a right-click menu with all of the above plus a figure-only "Toggle
  Interactive" entry. New IR fields `visible` / `lock_level` /
  `rotation` / `interactive` join the existing `z:` and `style {}`,
  with the same defaults (visible/interactive: true, no lock, no rot).
- New WASM bindings: `apply_figure_move` / `apply_figure_rotate` /
  `apply_figure_lock` / `apply_figure_visibility` /
  `apply_figure_interactive` / `apply_figure_delete` /
  `get_figure_hint_value`, mirroring the note / plot reverse-sync API
  shape.
- New IR helpers: `figure_world_extents_indexed` (aelm-render) and
  `build_figure_items` (aelm-render-interactive). The WASM
  `interactive_items` projection now extends figure items so the
  WebView actually sees them.

### Changed

- The renderer honours `visible: false` on figures by suppressing the
  geometry. The InteractiveItem projection still emits a small marker
  bbox at the figure's anchor so authors can flip visibility back on.

### Fixed

- **SVG / PNG exports no longer clip notes and figures** anchored
  outside the component bbox. `aelm-wasm`'s previous
  `compute_bounds_from_layout` ignored wires / notes / figures; it now
  shares the unified `compute_world_bounds` implementation with the
  CLI exporter.
- **SVG export renders at readable size.** `width` / `height`
  attributes now use 15 px/mm instead of mm units, so exported SVGs
  display at a practical size in browsers and Markdown previews.
- **SVG / PNG save dialog pre-fills the file name** from the active
  `.aelm` document (e.g. `amplifier.aelm` → `amplifier.svg`).
- **Triangle vertex handle aligned with apex on odd widths (#202).**
  Triangles with odd `size` widths (e.g. `triangle size: (3, 2)`) place
  their apex at a half-pitch position, but the interactive handle was
  computed with integer division (`w / 2`) and landed one half-pitch
  to the left of the drawn vertex. The vertex-offset helper now uses
  `f64` arithmetic so the handle bbox sits dead-centre on the apex,
  matching where the apex is actually rendered.

## [0.4.0] - 2026-05-09

### Added

- **Plot annotation layer (`plots {}`)**. Schematics can now carry data
  plots inline. Three trace sources are supported: function expressions
  (`y = sin(2*pi*1000*x)`) evaluated by a small Pratt parser, CSV files
  (`source: "data/scope.csv"` with named or indexed columns), and inline
  sample arrays (`samples: [(x, y), ...]`). Features: dual Y axis
  (`y2: { ... }` + per-trace `y2: true`), legend with auto-cycling 6-color
  palette, axis labels with optional display `scale:` factor, dashed
  grid lines per axis, and LTTB decimation (default 2000 points; 100k-row
  CSV captures stay snappy). Plot data is evaluated once per layout
  artifact so pan / zoom never re-runs the math layer or re-parses CSV.
  Diagnostics: `E_PLOT_001` (duplicate id), `E_PLOT_RANGE` (missing
  `x.range`), `E_PLOT_003` (unknown pin anchor), `E_PLOT_004` (bad
  expression). New WASM API `list_plots(ir, files)` reports per-plot
  metadata including the post-decimation point count. See
  `examples/plot-showcase.aelm`, `examples/csv-plot.aelm`, and the
  [DSL Plot wiki page](https://github.com/alphaelements/aelm/wiki/en-DSL-Plot).
- **Plot CSV traces in WebView** (#192 follow-up). The WASM render
  pipeline now threads a `plot_files` map alongside the layout result,
  so `source: "data/scope.csv"` traces render in the VSCode canvas
  exactly as they do in `aelm export-svg`. The VSCode extension scans
  the active document for `source:` paths and reads them off disk
  next to the parser's existing `use`-statement file map.
- **Plot inline + external style** (#192 follow-up). Plots cascade
  three style layers: engine defaults → external `plot { ... }`
  pseudo-element from `.astyle` → inline `style { ... }` block on the
  plot itself. New inline keys: `border_width`, `grid_color`,
  `text_color`. The astyle pseudo-element reuses the standard
  `fill_color` / `color` / `width` / `sub_color` / `font_color` /
  `font_size` properties.
- **Plot interactivity** (#192 follow-up). Plots are now movable and
  lockable from the VSCode WebView. Drag rewrites `place:`, the right-
  click Lock Level menu writes `lock:` inside the plot block body. New
  WASM APIs: `apply_plot_move`, `apply_plot_lock`. Rotation and mirror
  remain disabled — plot frames are axis-aligned. Implementation
  follows the new `add-interactive-element` skill that codifies the
  Note (#189) interactive pattern as a reusable recipe.
- **Plot samples raw block form** (#192 follow-up). `samples: [...]`
  now also accepts CSV-style bare `x, y` rows separated by newlines,
  alongside the original `(x, y)` paren form. Both forms can be mixed
  inside the same block, so authors can paste captured data without
  reformatting it.

- **Annotation layer**. Schematics can now carry text annotations and
  decoration shapes:
  - `notes { N1: note "..." }` blocks, plus inline `R1: ... note: "..."`
    on instances and `-> ... note: "..."` on connections. Block-form
    notes accept `place:` (absolute, pin-relative, or component-
    relative), `rot:`, `lock:`, `visible:`, and an inline `style { }`
    block that controls font_size, fill, font_weight, font_style,
    background, and per-line text_align. Multi-line bodies use a raw
    bracket form `note [ ... ]` — each source line becomes one
    rendered row. The renderer draws each note as a sticky-note memo
    card with a folded top-right corner and a theme-aware palette
    (light pale-yellow / dark deep-slate); `style { background: "none"; }`
    opts out of the card so text floats on the schematic. Hidden
    notes (`visible: false`) emit a small folded-page placeholder
    icon at the anchor.
  - `figures { ... }` block carrying decoration shapes: line, rect,
    circle, ellipse, arc, arrow, and text. Coordinates are integer
    grid cells with absolute or pin-relative anchors. Inline
    `style { stroke_color; stroke_width; fill; font_size; ... }`
    cascades over engine defaults; numeric `z:` controls stacking
    order. DRC and routing ignore the block. See
    `examples/figure-showcase.aelm`.

  Notes are interactive on the VSCode canvas with the same gesture
  set as part / module instances: drag to reposition, `R` rotates,
  `L` toggles lock, `Del` deletes, `V` toggles visibility, and right-
  click opens a Rotate / Visibility / Lock Level / Delete menu. The
  card, dog-ear, and glyphs rotate together; `lock:` blocks the drag
  write-back. New stylesheet `note { ... }` pseudo-element + `aelm
  apply-note-{move,rotate,lock,delete,toggle-visibility}` CLI
  subcommands. See `examples/note-showcase.aelm` and the
  [DSL Notes wiki page](https://github.com/alphaelements/aelm/wiki/en-DSL-Notes).

- **Inline math layer (`$...$`)**. Text inside a note body bracketed
  by `$ ... $` is now typeset as math instead of rendered verbatim.
  The new `aelm-math` crate ships a build-time glyph catalogue
  extracted from the SIL OFL-1.1 KaTeX TrueType fonts (italic Latin
  variables, upright digits / operators / brackets / punctuation,
  full Greek upper + lower with variants, ~50 relations and binary
  operators, big operators with stacked / right-side limits, blackboard-
  bold, and Caligraphic capitals — 393 glyphs at ~672 KB of Rust
  source). The schematic backend tessellates each glyph into a
  `RenderCommand::FilledPath` (new variant) — SVG emits a `<path>`
  with `fill-rule="evenodd"`, the wasm Canvas backend uses
  `Path2D` + `fill('evenodd')` via `CanvasWindingRule::Evenodd`, and
  the native PNG backend uses `tiny_skia::FillRule::EvenOdd`, so
  counters punch real holes consistently across all three render
  surfaces. Supported
  constructs: fractions, square roots, scripts, big operators with
  limits, absolute value, matrices (`pmatrix` / `bmatrix` / `vmatrix`
  / `Vmatrix` / `matrix`), implicit multiplication, and ~80 LaTeX
  command aliases. Note bbox + card geometry use the math layout's
  measured ascent / descent so fractions / sums / exponents stay
  enclosed by the viewBox. See `examples/math-showcase.aelm` and the
  [DSL Math wiki page](https://github.com/alphaelements/aelm/wiki/en-DSL-Math).
  OFL-1.1 attribution is mirrored in `THIRD_PARTY_NOTICES.md`,
  `vscode-extension/THIRD_PARTY_NOTICES.md`, and
  `web/renderer/THIRD_PARTY_NOTICES.md`.

### Fixed

- **Net-tap instances accepted as annotation anchors**. `notes {}`,
  `figures {}`, and `plots {}` placement (`place: G1.pin(...)`) now
  correctly resolve net-tap instances (`V1: net VCC`, `G1: net GND`).
  Previously they emitted spurious `E_NOTE_003` / `E_FIG_003` /
  `E_PLOT_003` errors.
- **Multiline note drag does not inject spurious `place:`**. Dragging a
  multiline `note [ ... ]` that contains a blank line no longer inserts
  a `place:` directive inside the bracket body. Both `locate_block_note`
  and `update_note_place_line` now track `[ ]` bracket depth.
- **Plot CSV WebView render** (#192). VSCode WebView no longer renders
  CSV-source plots as empty polylines. The fix routes
  `parse_with_imports*` file content through the layout response into
  the render pipeline so CSV bytes reach `evaluate_plots` regardless
  of which entry point (`render` / `export_svg` / `render_to_svg`)
  the host calls.
- **Math — spacing commands** (`\,`, `\;`, `\:`, `\!`, `\quad`,
  `\qquad`) now insert the correct amount of horizontal space instead
  of being silently ignored.
- **Math — relation padding**. Binary relations (`=`, `\le`, `\ge`,
  `\approx`, …) now carry 0.15 em padding on each side, matching
  LaTeX thin-space convention.
- **Math — fraction bar alignment**. The vinculum is drawn at the
  math axis (half x-height above the baseline) with proper clearance
  above numerator / below denominator; nested fractions no longer
  drift from the axis.
- **Math — mixed text + math baseline**. Text runs and math fences
  on the same note line now share the alphabetic baseline instead of
  each segment snapping to its own vertical origin.
- **Math — sqrt sizing**. Square roots now use continuous KaTeX/RaTeX-
  style scaling with a glyph-aware size cache, producing tighter fits
  on nested radicals.
- **Math — auto-sized delimiters**. Parentheses, brackets, and bars
  on matrices and absolute-value fences scale to the content height
  using ReX-style math-axis centering.
- **Math — VRule bar height**. Absolute-value bars now span only the
  actual content ascent/descent instead of forcing axis-symmetric
  height, so `|\frac{1}{\frac{2}{3}}|` no longer extends the top bar
  beyond the numerator.
- **Math — integral italic correction**. `\int` and friends apply an
  italic correction to prevent the upper limit from colliding with the
  integral sign's top curve.

### Changed

- **Math — performance**. `FilledPath` render commands are replaced by
  `MathGlyph` references with cached tessellation, eliminating
  redundant outline expansion on every render frame.

### Internal

- **Stdlib SVG snapshot regression suite**. New
  `examples/stdlib-gallery.aelm` (21 stdlib parts on one
  deterministic page) plus `aelm-render` snapshot tests under
  `crates/aelm-render/tests/snapshots/`. Pin position, label anchor,
  and viewBox regressions surface as a CI-visible diff. Update
  intentional changes with `INSTA_UPDATE=always cargo test -p
  aelm-render --test snapshot`. See `wiki/Reference/Dev-Environment.md`.

## [0.3.4] - 2026-05-06

### Fixed

- **CI — `build-wasm` now runs `scripts/build-wasm.sh`** (wasm-bindgen +
  wasm-opt) instead of bare `cargo build`. The previous job only produced
  the raw `.wasm` binary; webpack's `CopyPlugin` silently skipped the
  missing `wasm/` directory (`noErrorOnMissing: true`), so every published
  `.vsix` shipped without the WASM JS-glue files.
- **CI — `build-vsix` now compiles and bundles `aelm-lsp`** before
  packaging. The LSP server binary was never built in CI, so the installed
  extension always failed to start with `ENOENT` on Linux / WSL
  ([#5](https://github.com/alphaelements/aelm/issues/5)).

### Changed

- **Public repo (`aelm/`) relicensed to MIT** with a User-Created Works
  clause clarifying that `.aelm` / `.alib` files and their generated
  outputs are owned by their respective authors.
- **Public `aelm/` README** rewritten with logo, Marketplace badge,
  VSCode preview screenshot, and tables for examples and tips.

## [0.3.3] - 2026-05-05

### Fixed

- **CI — `test` job now refreshes the `aelm.wiki` submodule** before
  running `cargo test --all`. The shell-executor runner reuses its
  workspace across jobs, so a stale wiki checkout from a prior tag
  pipeline could leave the `feature_catalog_drift` integration test
  reading an outdated `Language constructs` table and panicking with
  *"no matching row in `aelm.wiki/en-Feature-Versions.md`"* even when
  the parent repo's submodule pointer was correct. The job now sets
  `GIT_SUBMODULE_STRATEGY: recursive` and explicitly runs `git
  submodule sync --recursive` + `git submodule update --init
  --recursive --force aelm.wiki` so every test run sees the SHA
  recorded in the current commit. Root-caused on the v0.3.2 tag
  pipeline (#217) where this masquerade prevented the release jobs
  from ever starting (no Marketplace publish, no GitHub release).
  v0.3.3 is functionally identical to v0.3.2 from a user perspective —
  the bump exists solely to actually ship the v0.3.2 changes through
  the now-fixed pipeline.

## [0.3.2] - 2026-05-05

### Added

#### DSL & language

- **Sigil supply names** — net names in `nets {}` and `<id>: net N`
  taps may now begin with `+` or `-`, so authors can declare rails by
  their schematic-conventional names (`+5V`, `+3V3`, `-12V`). The
  sigil is part of the name and is preserved verbatim through parse,
  IR, layout, and the rendered label. Existing identifier rules
  still apply elsewhere — pin / instance / part names are unchanged.

- **Net taps** (#187) — `<id>: net <NetName>` instances inside the
  existing `instances {}` block let one declared net carry as many
  visual flag taps as the schematic needs, all sharing the *net name*
  on screen. Eliminates the `GND1` / `GND2` / `GND3` instance-id label
  pollution that pre-0.3.1 schematics inherited from forcing a unique
  identifier per power / GND symbol position. The new `nets { N: type
  { symbol: <template>, label_only: <bool> } }` body chooses the tap's
  symbol (defaults: `gnd_flag` for ground, `power_flag` for power).
  Taps render through the regular layout / routing / rendering
  pipeline and inherit the same `place: / rot: / mirror: / lock: /
  style {} / #G1` astyle semantics as any other instance.

#### Stdlib & symbols

- **DMM (Digital Multimeter)** — new `dmm` builtin symbol plus
  `part DMM { hi: passive @1; lo: passive @2 }` in
  `measurement.alib`. Front-panel rectangular instrument with an LCD
  readout and `+` / `−` jack labels — distinct from the textbook
  circle-with-V `Voltmeter` so tutorials and tips can draw a real
  bench-top instrument.

#### Rendering

- **Component values rendered next to the reference designator** —
  instances declared with `value:` (e.g. `Resistor(value: 1k)`,
  `Capacitor(value: 1.6n)`, `VAC(amplitude: 100m, frequency: 1k)`)
  now show that primary param as a second text block on the
  schematic, point-symmetric to the reference designator across the
  symbol body center. Picks `value` first, falling back to the first
  declared param, so single-param sources like `VDC(voltage: 5)` and
  multi-param sources like `VAC(...)` both display the most-defining
  quantity.

#### DRC

- **`W_NET_TAP_ORPHAN`** (#187) — flags a `<id>: net X` tap with no
  incoming connection on either `<id>.pin` or the bare net name `X`.
- **`W_NET_UNUSED`** (#187) — flags a `nets { X: ... }` declaration
  with no taps and no pin references.
- **`E_NET_TAP_UNKNOWN_NET`** (#187) — error when a tap references a
  net not declared in the surrounding `nets {}` block.

### Fixed

- **Routing wire-overlap demoted from Error to Warning** — rotating
  / mirroring a multi-pin connector can leave the Steiner-tree router
  with no overlap-free path on tightly packed schematics. The
  diagnostic now emits a Warning so the canvas still renders, with
  the user free to disambiguate via `via:` waypoints.
- **Pin-relative placement now waits for the anchor instance to
  resolve** — `place: <other>.pin(dx, dy)` no longer reads the
  force-layout seed of an unresolved anchor, so chained placements
  like `R1: place: V1.pin(0, 3)` land at the author's coordinates
  regardless of declaration order.
- **CST lookup uses exact-name match** — `find_node_by_kind_and_name`
  now requires the search name at a definition position
  (`<name>:` or `<keyword> <name>`), not anywhere as a substring.
  Prevents the case where dragging a multi-unit IC sub-instance
  silently moved a sibling whose `place:` anchor mentioned the same
  token (e.g. `Cb1: place: U1B.inp_b(...)` masquerading as a `U1B`
  declaration).
- **Reverse sync tolerates aligned whitespace around `->` / `==`** —
  author-aligned lines like `R1.p1   -> R2.p2` now match for
  duplicate-add / delete / waypoint-update, and the writer preserves
  the alignment when rewriting.
- **Component label fallback for R90 / R270** — text anchor flips
  to `start` / `end` so glyphs extend away from a rotated body's
  outline instead of crowding it; the point-symmetric value emit
  picks up the flipped anchor too.
- **DRC supply-pin model unified across power and ground** (#187) —
  `DRC-E003` removed; `DRC-E002` now covers both supply directions.
  Net-tap presence suppresses `W_NET_001` / `W_NET_002` on supply
  nets, since the tap is the visible connection.

## [0.3.1] - 2026-05-03

Republish of the v0.3.0 release. The npm `@alphaelements/aelm-renderer@0.3.0`
package was published from an older snapshot before the v0.3.0 tag was cut,
so its contents diverged from the VS Code Marketplace and GitHub Release
artifacts that carry the correct v0.3.0 toolchain. npm's 72-hour unpublish
window had already closed, so this release re-cuts the same toolchain under
v0.3.1 to put a consistent build on every distribution channel. No
functional changes since v0.3.0 — see [0.3.0] below.

## [0.3.0] - 2026-05-03

The Cargo workspace, VSCode extension, and `@alphaelements/aelm-renderer`
move together to 0.3.0. Manifests were bumped earlier (chore(release)
on 2026-04-27); the tag was held pending the changelog catch-up
captured in this section. Per-store changelogs
([`vscode-extension/CHANGELOG.md`](./vscode-extension/CHANGELOG.md),
[`web/renderer/CHANGELOG.md`](./web/renderer/CHANGELOG.md)) expand on
the user-facing impact.

### Added

#### DSL & language

- **Per-instance pin name override** (#182):
  `J1: Connector4 pins: { pin_1: vcc, pin_2: gnd, pin_3: tx, pin_4: rx }`
  renders semantic labels inside the symbol body, auto-widens the body
  to fit, and resolves both alias (`J1.vcc`) and canonical pad
  (`J1.pin_1`) names in `connections { ... }`.
- **Per-instance pin label visibility** (#183): `show_pin_labels: true|false`
  on an instance overrides the template's static `show_label`.
- **`nc:` no-connect hint** (#186): `nc: all` / `nc: [pin1, pin2]`
  suppresses DRC-E001 for the targeted pins. DRC-E001 itself is now a
  Warning (was Error) so unintentional unconnected pins still surface
  but no longer fail builds.
- **`# aelm-version:` semver header** (#175): replaces the legacy
  `# aelm-syntax: 1` integer header. The parser checks it on every
  file; missing/unsupported headers are now Errors (#180).
- **`Feature` enum + `introduced_in()` catalog** (#175 / #180): every
  DSL construct carries a "version-in-which-it-appeared" record, used
  to gate features against the file's declared `# aelm-version:`.

#### Renderer / canvas

- **NC mark rendering** (#186): ✗ symbol drawn at the stub tip of every
  pin marked `nc:` that has no wire attached. New `RenderCommand::NoConnect`
  with both SVG and Canvas backend support.
- **Connector body-size parity fix**: layout resolves symbols via part
  name (matching the render path), so connector bodies and NC marks
  agree on geometry.
- **Perpendicular pin labels** rotate correctly on left/right edges so
  long names always read outward and never cross the body.
- **`AntennaMonopole` / `PowerArrow` label anchors** no longer drift
  from the symbol at non-default rotations.

#### CLI

- **`aelm render`** (#174 / #131) — unified subcommand replacing
  `export-svg` (kept as a thin alias).
- **`aelm render --format png`** (#174) — PNG output via the new
  `PngBackend`. `--embed-font <path>` produces self-contained PNGs.
- **`aelm migrate`** (#175) — adds `# aelm-version:` headers to legacy
  files in-place.
- **`aelm check --strict-feature-version`** (#180) — fails when a file
  uses a feature not yet introduced in its declared version.

#### Editor / LSP

- **DRC integration into the webview pipeline** (#185): DRC runs inside
  `runPipeline()`, surfacing design-rule diagnostics in Problems.
- **Context-aware completions**: `nc:` pin lists (#186),
  `show_pin_labels: true|false` (#183), `# aelm-version:` diagnostics
  surfaced in editor and webview (#180).

#### Renderer npm package (`@alphaelements/aelm-renderer`)

- **Initial public release as a separate npm package** (#174 / #129 /
  #130): `renderToSvg` / `renderToCanvas` one-shot APIs, plus
  cached `buildLayout` / `renderLayoutToCanvas` for hot-path pan/zoom
  (BD-Pipeline-Layering). Built from the same Rust workspace as the
  VSCode extension via `cargo build … --no-default-features --features
  embedded-stdlib` (MIT bundle).
- Webview now routes through the unified WASM `CanvasBackend`; the
  legacy JS canvas-bridge dispatcher is removed (#174).

#### Stdlib

- **Phase 4.5 stdlib expansion** (#149-#151): bundled stdlib provides
  ready-to-use parts for every builtin symbol (97 templates), plus
  matching `examples/*-gallery.aelm` showcase files.

### Changed

- Distribution terminology consolidated to **"MIT bundle"** throughout
  code comments, specs, and renderer package metadata. The workspace
  remains Open Core — Rust sources are Proprietary; only the published
  binary is MIT-licensed.
- `aelm-render-interactive` carved out of `aelm-render` so the core
  renderer crate stays pipeline-only (#174 P5.5).
- `aelm export-svg` and `aelm export-png` are now thin aliases for
  `aelm render` / `aelm render --format png`.

### Fixed

- WASM OSS build: `cfg_attr` allow attributes now carry the required
  `reason` field (#178).
- SVG output: `<svg>` root carries explicit `width` / `height`;
  zero-length pin stubs are skipped; `StrokeStyle::solid` / `::dashed`
  default to round caps for SVG/Canvas parity (#174).
- CLI `aelm render` and WASM `render_to_svg` exclude the background
  grid by default (#174).
- VSCode F5: WASM now builds before webpack so the first launch picks
  up renderer-side changes.

### Internal

- **Public API surface gates** (#175 prep, six layers):
  - Layer 1: `cargo-semver-checks` pre-push gate for publishable Rust
    crates.
  - Layer 2: committed `web/renderer/api-snapshot.d.ts` + drift gate
    for the WASM TS surface.
  - Layer 3: `tests/cli-snapshots/*-help.txt` snapshots for every
    `aelm` subcommand.
  - Layer 4: `tests/conformance/syntax-v1/` frozen corpus the parser
    must accept across releases.
  - Layer 5: built-in symbol snapshot tests covering all 97 templates.
- **Lefthook gates added**:
  - `version-sync` (manifest drift across the three lockstep manifests).
  - `renderer-changelog` (any change to `aelm-ir` / `aelm-parser` /
    `aelm-symbol` / `aelm-render` / `aelm-wasm` requires a
    `web/renderer/CHANGELOG.md` entry, with `Renderer-Impact: none`
    trailer as the explicit escape hatch).
  - `feature-version-consistency` (#180): every DSL `Feature` variant
    must have an `introduced_in` entry.
- **CI**:
  - `verify-tag-matches-version` checks the renderer manifest in
    addition to Cargo + extension.
  - `Feature::introduced_in()` is gated against the latest published
    tag so a feature can never claim an unreleased version.
  - `binaryen` (wasm-opt) installed in the renderer build job for size
    optimization on tagged releases.
  - n8n webhook notification on successful marketplace publish.
- **`knope` integration** for Conventional-Commits-driven release notes
  (`knope.toml`); root `CHANGELOG.md` is the authoritative target,
  per-store CHANGELOGs remain hand-maintained.
- **Cargo workspace + extension + renderer manifests bumped to 0.3.0**
  (`chore(release)`); tag pending.

## [0.2.1] - 2026-04-27

This is the first release in which the renderer npm package is part of
the toolchain lockstep. The Cargo workspace, the VSCode extension, and
`@alphaelements/aelm-renderer` now share one MAJOR.MINOR.PATCH and are
bumped together by `scripts/release.sh`.

### Added

- `@alphaelements/aelm-renderer@0.2.1` joins the toolchain (renamed
  from `@aelm/renderer`; the `aelm` npm scope is unavailable due to
  typosquat-prevention against the existing `elm` package).
- Aelm Compatibility Matrix (`en-Compatibility` / `ja-Compatibility`)
  documenting which versions of the four artifacts ship together.
- DSL syntax versioning: `# aelm-syntax: 1` header is now the contract
  for `.aelm` and `.alib` files; the parser checks it and `aelm
  migrate <file>` adds it to legacy files.
- Public API surface verification — `cargo-semver-checks` for the
  publishable Rust crates, a committed
  `web/renderer/api-snapshot.d.ts` for the WASM TS surface, and
  `tests/cli-snapshots/*-help.txt` for every `aelm` subcommand.
- Built-in symbol library snapshot tests
  (`crates/aelm-symbol/tests/builtin_snapshots.rs` covers all 38
  templates) so any change to pin layout / dimensions / electrical
  defaults must be acknowledged in the CHANGELOG.
- Conformance corpus at `tests/conformance/syntax-v1/` — a frozen set
  of `.aelm` fixtures every release build must parse and render.

### Changed

- `scripts/release.sh` and CI `verify-tag-matches-version` now operate
  on three manifests instead of two; the renderer cannot drift.
- Distribution terminology: "OSS bundle" replaced by "MIT bundle"
  throughout code comments and specs. Aelm follows Open Core — Rust
  sources are Proprietary, only the published binary is MIT-licensed.

### Internal

- New lefthook pre-push gates: `version-sync` (manifest drift) and
  `renderer-changelog` (renderer-relevant crate changes require a
  CHANGELOG entry, with a `Renderer-Impact: none` trailer escape).
- `knope` integration for automated release-note generation from
  Conventional Commits; `knope.toml` at repo root defines the
  package-wide workflow.
- New spec: [`wiki/Specification/BD-Versioning.md`](./wiki/Specification/BD-Versioning.md)
  records the lockstep policy, the six compatibility layers, and how
  each is verified.

[Unreleased]: https://github.com/alphaelements/aelm/compare/v0.5.0...HEAD
[0.5.0]: https://github.com/alphaelements/aelm/releases/tag/v0.5.0
[0.4.2]: https://github.com/alphaelements/aelm/releases/tag/v0.4.2
[0.4.1]: https://github.com/alphaelements/aelm/releases/tag/v0.4.1
[0.4.0]: https://github.com/alphaelements/aelm/releases/tag/v0.4.0
[0.3.4]: https://github.com/alphaelements/aelm/releases/tag/v0.3.4
[0.3.3]: https://github.com/alphaelements/aelm/releases/tag/v0.3.3
[0.3.2]: https://github.com/alphaelements/aelm/releases/tag/v0.3.2
[0.3.1]: https://github.com/alphaelements/aelm/releases/tag/v0.3.1
[0.3.0]: https://github.com/alphaelements/aelm/releases/tag/v0.3.0
[0.2.1]: https://github.com/alphaelements/aelm/releases/tag/v0.2.1
