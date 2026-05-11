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

### Added

- **Figure interactivity (#196).** The `figures {}` block now lights up the
  same WebView gestures notes have had since #189: click selection with
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

[Unreleased]: https://github.com/alphaelements/aelm/compare/v0.4.0...HEAD
[0.4.0]: https://github.com/alphaelements/aelm/releases/tag/v0.4.0
[0.3.4]: https://github.com/alphaelements/aelm/releases/tag/v0.3.4
[0.3.3]: https://github.com/alphaelements/aelm/releases/tag/v0.3.3
[0.3.2]: https://github.com/alphaelements/aelm/releases/tag/v0.3.2
[0.3.1]: https://github.com/alphaelements/aelm/releases/tag/v0.3.1
[0.3.0]: https://github.com/alphaelements/aelm/releases/tag/v0.3.0
[0.2.1]: https://github.com/alphaelements/aelm/releases/tag/v0.2.1
