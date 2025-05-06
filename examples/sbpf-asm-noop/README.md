# sBPF No-Operation Example

[Home](../../README.md) > [Examples](../README.md) > No-Operation Example

---

This example demonstrates the minimal structure for a Solana BPF program written in assembly. The program simply returns success (0) without performing any operations.

## Project Structure

```
sbpf-asm-noop/
├── deploy/             # Deployment scripts and artifacts
├── src/
│   └── noop/           # Source code for the no-operation program
└── tests/              # Test files for the program
```

## Source Code

The main program is located in [src/noop/noop.s](./src/noop/noop.s) and consists of a simple entrypoint function that immediately returns.

## Building the Program

To build this example:

```bash
# Assuming you have the sBPF toolchain installed as described in the Toolchain Setup guide
cd src/noop
make
```

This will compile the assembly code and generate the program ELF file.

## Deploying to Solana

Deployment instructions can be found in the [deploy](./deploy) directory.

## What to Learn Next

After understanding this minimal example:

1. Check out the [Fibonacci Example](../solana-fibonacci-asm/) to see a more complex program
2. Review the [Execution Model](../../docs/foundations/02_execution_model.md) to understand how the program runs on Solana
3. Look at the [Register Reference](../../docs/appendices/05_register_reference.md) for details on register usage

---

| [← Back to Examples](../README.md) | [Next: Fibonacci Example →](../solana-fibonacci-asm/) |
|:-----------------------------------|----------------------------------------------------:| 