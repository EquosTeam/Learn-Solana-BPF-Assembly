# Compiler and Linker Flags Reference

This appendix documents common compiler and linker flags used for building sBPF programs, primarily when compiling from C or similar languages. Assembly projects might bypass some compiler flags.

## Clang Compiler Flags (when compiling C/C++ to sBPF)

| Flag | Purpose |
|------|---------|
| `-target bpfel` | Target eBPF in little-endian format. `bpfel-unknown-none` is also commonly used. |
| `-march=sbf` | Specify Solana BPF architecture extension. Enables sBPF features and verifier rules. |
| `-mcpu=sbfv1` / `-mcpu=sbfv2` | Optional: Explicitly target sBPFv1 or v2 instruction set. Often implied by `-march=sbf`. |
| `-O2` / `-Os` / `-Oz` | Optimization level. `-O2` is common, `-Os` (size) or `-Oz` (more size) often preferred for deployment cost. |
| `-fno-builtin` | Disable recognition of standard library functions as builtins. |
| `-fno-jump-tables` | Avoid jump tables, which are not directly supported in sBPF. |
| `-fno-stack-protector` | Disable stack canaries (not applicable to sBPF environment). |
| `-fomit-frame-pointer` | Allows the compiler to potentially use the frame pointer register (R10) for other purposes if beneficial (though R10 is read-only in sBPF). Avoid for debugging. |
| `-g` | Generate debug information (DWARF). Useful for debugging but should be stripped for deployment. |
| `-c` | Compile only, do not link. Generates object files (`.o`). |
| `-fPIC` | Generate Position Independent Code. Often not strictly necessary for simple static sBPF programs but might be required in complex scenarios or by default toolchain settings. |

## Common Compilation Command (C to Object File)

```sh
clang \
  -target bpfel \
  -march=sbf \
  -O2 \
  -fno-builtin \
  -fno-jump-tables \
  -fno-stack-protector \
  -c \
  -o program.o \
  program.c
```

## Linker (ld.lld) Flags

| Flag | Purpose |
|------|---------|
| `--image-base 0x100000000` | **Crucial:** Set the base virtual address for the program text segment. |
| `--entry entrypoint` | Set program entry point symbol (e.g., `entrypoint`). |
| `-T <script.ld>` | Specify the linker script to control section layout. |
| `-shared` | Create a shared object (`.so`) file, the format expected by Solana. |
| `--Bstatic` | Link statically, do not use dynamic libraries. |
| `-z notext` | Allow relocations in the text segment (sometimes needed, especially with rodata). |
| `-z noexecstack` | Mark stack as non-executable (security best practice). |
| `--gc-sections` | Enable garbage collection of unused sections (requires appropriate section setup). Often used with `-ffunction-sections -fdata-sections` during compilation. |
| `--strip-all` / `--strip-debug` | Remove symbol table and/or debug sections to reduce final binary size for deployment. |
| `-o <output.so>` | Specify the output file name. |

## Common Linking Command (Object File to SO)

```sh
ld.lld \
  --image-base 0x100000000 \
  --entry entrypoint \
  -T link.ld \
  -shared \
  --Bstatic \
  -z notext \
  --strip-all \
  -o program.so \
  program.o
```

## Flag Considerations

### Version-Specific Options

- Ensure compiler flags (`-march`/`-mcpu`) match the targeted sBPF version (v1 or v2).
- Do not use instructions specific to one version if targeting the other.

### Optimization Levels

- `-O0`: No optimization (debugging).
- `-O1`: Basic optimizations.
- `-O2`: Good balance of size/performance.
- `-Os`/`-Oz`: Optimize for size (recommended for deployment).

### Debugging

- Compile with `-g`.
- Link *without* `--strip-all` or `--strip-debug`.
- Consider `-O0` or `-O1` and avoid `-fomit-frame-pointer`.

## Common Errors and Solutions

| Error | Likely Cause | Solution |
|-------|-------------|----------|
| "Relocations in read-only section" | Missing `-z notext` during linking. | Add `-z notext` to `ld.lld` flags. |
| "Undefined symbol: entrypoint" | Entry function missing or not global / Linker `--entry` mismatch / Script `ENTRY()` mismatch. | Ensure entry function exists and matches linker flags/script. |
| "Unknown CPU type" | Incorrect `-march` or `-mcpu` value. | Use `sbf`, `