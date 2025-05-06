# Tool-chain & Environment

Solana's official tooling bundles a patched LLVM that understands the
`+solana` eBPF feature-set, but you can also use any recent upstream LLVM
(≥ 14) with the `bpfel` target.

## Clang in one line

```bash
clang -Os -target bpfel -march=bpfel+solana \
      -c src/hello.s -o build/hello.o
```

Important flags:

| Flag | Purpose |
|------|---------|
| `-target bpfel` | little-endian BPF. |
| `-march=bpfel+solana` | Enable Solana-specific verifier rules. |
| `-Oz / -Os` | Optimise for size (smaller = cheaper deploy fee). |
| `-g0` | Strip debug for deploy (optional). |

## Linking

Use `ld.lld` (or `clang -fuse-ld=lld`) with a **custom linker script** that:

1. Places `.text` at `0x1000_0000`.
2. Sets the entry symbol via `ENTRY(e)`.
3. Optionally discards unused sections to save bytes.

Example (trimmed) linker script from the Fibonacci sample:

```ld
SECTIONS {
  . = SIZEOF_HEADERS;
  .text    : { *(.text*)   } : text
  .rodata  : { *(.rodata*) } : text
}
ENTRY (e)
```

Link command:

```bash
ld.lld -z notext --image-base 0x100000000 -shared \
       -T fib.ld -o fib.so build/fib.o
```

## `cargo build-sbf` vs manual

The Solana SDK command `cargo build-sbf` compiles **Rust** crates to sBPF.  When
working with raw assembly you usually bypass it and invoke Clang/Lld directly
(see `examples/sbpf-asm-noop/Makefile`).

## Directory layout suggestion

```
docs/            # This guide
src/             # .s or .c files
build/           # .o intermediates
deploy/          # final .so + keypair.json
Makefile         # wrapper around clang/ld
```

---

| [← Previous: Execution Model](./02_execution_model.md) | [Up: Foundations](./README.md) |
|:------------------------------------------------------|:------------------------------:| 