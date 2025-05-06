[Home](../../README.md) > [Foundations](./README.md) > Introduction to sBPF

---

# Introduction to sBPF on Solana

Solana **sBPF** (often written *SBF* inside Solana sources) is Solana's custom
variant of the extended Berkeley Packet Filter (eBPF) instruction-set and
virtual-machine that runs every on-chain program.

Why Solana uses its own flavour of BPF:

* **Determinism** – the Solana runtime must produce identical results on every
  validator.  Kernel-level helpers such as maps or kfuncs that rely on host
  state are therefore removed.
* **Performance** – BPF is a compact RISC-like ISA with a verifier that allows
  the JIT compiler to translate byte-code into native machine code quickly.
* **Language-agnostic** – anything that can target LLVM's `bpf` backend can be
  compiled for Solana (Rust, C, C++, Assembly, Zig, etc.).
* **Security** – the byte-code is verified before execution, preventing
  undefined behaviour such as out-of-bounds memory access.

Writing straight **assembly** instead of relying on higher-level languages can
be useful when you need **maximum control over compute-units, code size, and
branch-level performance** or when you simply want to understand how Solana
works at the lowest level... hacking if fun.

If you have never touched BPF assembly before, don't worry, the following
chapters build up from the ground level: registers, memory model, instruction
syntax, and finally real-world examples such as a Fibonacci calculator written
in only a few dozen instructions.

## Next Steps

After understanding the basics of sBPF, continue to the [Execution Model](./02_execution_model.md) to learn how sBPF programs run on Solana.

For practical examples, check out the [No-Operation Example](../../examples/sbpf-asm-noop/) which demonstrates a minimal sBPF program.

---

| [← Back to Table of Contents](./README.md) | [Next: Execution Model →](./02_execution_model.md) |
|:-------------------------------------------|--------------------------------------------------:| 
