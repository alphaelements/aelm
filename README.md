# Aelm — Agent-native Electronic Layout Markup

Aelm is a text-based electronic circuit CAD tool that runs as a VSCode extension powered by Rust and WebAssembly.

## Features

- **Text-based schematic entry** — describe circuits in `.aelm` files
- **Live preview** — see your circuit rendered as you type
- **Standard parts library** — built-in resistors, capacitors, ICs, connectors, and more
- **Hierarchical design** — compose modules from sub-modules
- **Style system** — customize colors, fonts, and layout via `.astyle` files

## Installation

### From VS Code Marketplace

> Coming soon

### From GitHub Releases

1. Download the latest `.vsix` from [Releases](https://github.com/alphaelements/aelm/releases)
2. In VS Code: `Ctrl+Shift+P` → "Extensions: Install from VSIX..."
3. Select the downloaded `.vsix` file

## Quick Start

Create a file named `hello.aelm`:

```
part Resistor {
    pins {
        a: passive @1
        b: passive @2
    }
    symbol: resistor
}

module Hello {
    instances {
        R1: Resistor(value: 1k)
        R2: Resistor(value: 4.7k)
    }
    connections {
        R1.b -> R2.a
        R2.b -> R1.a
    }
}
```

Open the file in VS Code with the Aelm extension installed to see the circuit preview.

## DSL Overview

Aelm circuits are built from **parts**, **modules**, and **connections**:

- `part` — defines a component with pins and a symbol
- `module` — instantiates parts and connects them
- `->` — creates a connection (wire) between pins
- `import` — loads parts from `.alib` library files

See the [examples/](examples/) directory for more sample circuits.

## Examples

| File | Description |
|---|---|
| [hello.aelm](examples/hello.aelm) | Minimal circuit — two resistors |
| [transistor_circuit.aelm](examples/transistor_circuit.aelm) | Transistor amplifier |
| [power_symbols.aelm](examples/power_symbols.aelm) | Power and ground symbols |
| [all_symbols.aelm](examples/all_symbols.aelm) | All built-in symbol types |
| [hierarchy_test.aelm](examples/hierarchy_test.aelm) | Hierarchical module design |

## Reporting Issues

Found a bug or have a feature request? Please open an [issue](https://github.com/alphaelements/aelm/issues).

## License

Proprietary. See [LICENSE](LICENSE) for details.
