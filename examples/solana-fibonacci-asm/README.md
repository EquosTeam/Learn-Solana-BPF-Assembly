# Solana Fibonacci Calculator in sBPF Assembly

[Home](../../README.md) > [Examples](../README.md) > Fibonacci Calculator

---

This example demonstrates a Fibonacci number calculator written in sBPF assembly. The program accepts a parameter specifying which Fibonacci number to calculate and returns the result.

## Project Structure

```
solana-fibonacci-asm/
├── deploy/             # Deployment scripts and artifacts
├── src/
│   └── fib/            # Source code for the Fibonacci calculator
└── tests/              # Test files for the program
```

## Source Code

The main program is located in [src/fib/fibonacci.s](./src/fib/fibonacci.s) and demonstrates:

1. Parsing input parameters from the instruction data
2. Implementing the Fibonacci algorithm using registers and stack
3. Returning the calculated result

## Building the Program

To build this example:

```bash
# Assuming you have the sBPF toolchain installed as described in the Toolchain Setup guide
cd src/fib
make
```

This will compile the assembly code and generate the program ELF file.

## Deploying to Solana

Deployment instructions can be found in the [deploy](./deploy) directory.

## Algorithm Overview

The Fibonacci implementation uses an iterative approach:
1. Check if the input is 0 or 1, return the input directly if true
2. Otherwise, initialize `a=0` and `b=1`
3. Loop from 2 to n, calculating `c = a + b`, `a = b`, `b = c`
4. Return the final value of `b`

## What to Learn Next

After understanding this example:

1. Examine the [Token Program Example](../token.sbpf/) for a more complex implementation
2. Review the [Instruction Set](../../docs/reference/03_instruction_set.md) to understand all available sBPF instructions
3. Look at the [Cross-Program Invocation](../../docs/advanced/) documentation to learn about more advanced techniques

---

| [← Previous: No-Operation Example](../sbpf-asm-noop/) | [Next: Token Program Example →](../token.sbpf/) |
|:-----------------------------------------------------|------------------------------------------------:| 