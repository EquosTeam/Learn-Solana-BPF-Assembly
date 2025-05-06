# sBPF Assembly Examples

[Home](../README.md) > Examples

---

This directory contains practical examples of Solana BPF Assembly programs. Each example demonstrates different aspects of sBPF programming, from the simplest no-operation program to more complex functionality.

## Available Examples

### [No-Operation Program](./sbpf-asm-noop/)
A minimal Solana program written in assembly that simply returns success. Perfect for understanding the basic structure of an sBPF program.

### [Fibonacci Calculator](./solana-fibonacci-asm/)
An assembly program that calculates Fibonacci numbers. This example demonstrates:
- Basic arithmetic operations
- Control flow with jumps
- Function calls
- Parameter handling

### [Token Program](./token.sbpf/)
A more complex implementation of a token program in assembly. This demonstrates:
- Account data manipulation
- Error handling
- Advanced sBPF techniques

## Using These Examples

Each example directory contains:
- Source code in the `src` directory
- Build scripts
- Deployment instructions in the `deploy` directory
- Test files

Follow the README.md in each example directory for specific instructions on building, deploying, and testing.

## Learning Path

We recommend exploring these examples in order:
1. Start with the [No-Operation Program](./sbpf-asm-noop/) to understand the basic structure
2. Move on to the [Fibonacci Calculator](./solana-fibonacci-asm/) to learn arithmetic and control flow
3. Finally, explore the [Token Program](./token.sbpf/) for a complete, real-world implementation

## Related Documentation

- [Foundations](../docs/foundations/): Core concepts for understanding sBPF
- [Reference](../docs/reference/): Detailed instruction set and syntax guides
- [Appendices](../docs/appendices/): Additional reference materials 