# AELM — Agent-native Electronic Layout Markup

AELM is a text-based electronic circuit CAD tool that runs as a VSCode extension powered by Rust and WebAssembly. Define circuits using a dedicated DSL, render schematics automatically, and manage your designs with version control.

## Repository Structure

This is the private development repository. Public-facing content is managed via submodules:

- `aelm/` — [Public repo](https://github.com/alphaelements/aelm) (README, examples, releases)
- `aelm.wiki/` — [Public wiki](https://github.com/alphaelements/aelm/wiki) (reference documentation)

## Build

```bash
cargo test --all                                    # all tests
cargo clippy --all-targets -- -D warnings           # strict lint
cargo build --target wasm32-unknown-unknown -p aelm-wasm --release  # WASM
cd vscode-extension && npm run compile              # VSCode extension
```

## License

Copyright (c) 2026 Alpha Elements. All rights reserved.
