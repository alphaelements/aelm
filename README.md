# Aelm — Agent-native Electronic Layout Markup

Aelm is a text-based electronic circuit CAD tool that runs as a VSCode extension powered by Rust and WebAssembly. Define circuits using a dedicated DSL, render schematics automatically, and manage your designs with version control.

## Quick Start

```aelm
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

Copyright (c) 2026 Alpha Elements. All rights reserved.
