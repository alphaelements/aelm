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

- **Annotation layer — Phase 1 (notes)** (#188 / #189). Three author
  forms collapse onto one IR vec under `ModuleDef.notes`:
  - `notes { N1: note "text" place: ... visible: ... style { ... } }`
    — free-standing labels with absolute or pin-relative placement.
  - `R1: ... note: "..."` — short inline label on an instance,
    auto-positioned above the reference designator.
  - `-> ... note: "..."` — inline label on a connection, drawn at the
    routed wire's longest-segment midpoint.
  Hidden notes (`visible: false`) emit a tiny grey placeholder circle
  instead of text so the editor can show "there is a hidden note here"
  without re-parsing. Inline `style { ... }` and a new stylesheet
  `note { ... }` selector cascade onto the rendered text. Diagnostics:
  `E_NOTE_002` / `E_NOTE_003` / `E_NOTE_004` / `E_NOTE_005`. See
  `examples/note-showcase.aelm` and `Specification/BD-Annotation-Note`.

### Internal

- **Stdlib SVG snapshot regression suite** (#152). Added
  `examples/stdlib-gallery.aelm` (21 stdlib parts on a single
  deterministic page) plus `crates/aelm-render/tests/snapshot.rs`,
  which renders the gallery and one solo SVG per part through the
  same Stage 2 + Stage 3 pipeline as `aelm render` and asserts each
  output against an `insta` snapshot under
  `crates/aelm-render/tests/snapshots/`. Pin position, label anchor,
  and viewBox regressions now surface as a CI-visible diff. Update
  intentional changes with `INSTA_UPDATE=always cargo test -p
  aelm-render --test snapshot`; rejected snapshots ride out of the
  GitLab `test` job as `**/snapshots/*.snap.new` artifacts. See
  `wiki/Reference/Dev-Environment.md` for the full workflow.

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

[Unreleased]: https://github.com/alphaelements/aelm/compare/v0.3.0...HEAD
[0.3.0]: https://github.com/alphaelements/aelm/releases/tag/v0.3.0
[0.2.1]: https://github.com/alphaelements/aelm/releases/tag/v0.2.1
