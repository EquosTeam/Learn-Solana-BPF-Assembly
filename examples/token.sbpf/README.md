# sBPF Token Program Example

[Home](../../README.md) > [Examples](../README.md) > Token Program

---

This example demonstrates a complete token program implementation in sBPF assembly. The program supports basic token operations including minting, transferring, and burning tokens.

## Project Structure

```
token.sbpf/
├── TEST/               # Test scripts and fixtures
└── src/                # Source code for the token program
```

## Source Code

The Token Program demonstrates advanced sBPF programming techniques including:

1. Account data deserialization and validation
2. Complex control flow
3. Error handling
4. Cross-program invocation (CPI)
5. State management

## Key Features

The Token Program implements the following functionality:

- Token initialization
- Minting new tokens
- Transferring tokens between accounts
- Burning tokens
- Authority management
- Account validation

## Building the Program

To build this example:

```bash
# Assuming you have the sBPF toolchain installed as described in the Toolchain Setup guide
cd token.sbpf
make
```

## Testing

The TEST directory contains testing utilities and scripts. To run the tests:

```bash
cd TEST
./run_tests.sh
```

## What to Learn Next

After understanding this example:

1. Review the [Advanced Topics](../../docs/advanced/) section to learn more about complex sBPF programming techniques
2. Learn about deployment strategies in the [Usage Guide](../../docs/usage/)

---

| [← Previous: Fibonacci Calculator](../solana-fibonacci-asm/) | [Back to Examples →](../README.md) |
|:-------------------------------------------------------------|----------------------------------:| 