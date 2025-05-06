# sBPF GAS Syntax Primer

Solana BPF (sBPF) assembly typically uses a syntax similar to the GNU Assembler (GAS), specifically the AT&T syntax style, although tooling might vary. This primer covers the essential elements you'll encounter when reading or writing sBPF assembly.

## Basic Structure

-   **Sections**: Code usually resides within the `.text` section.
    ```assembly
    .section .text
        // Code goes here
    ```
-   **Labels**: Labels define locations in the code, often used as jump targets or function entry points. They end with a colon (`:`).
    ```assembly
    my_function:
        mov64 r1, r0
        ja exit_label

    exit_label:
        exit
    ```
-   **Global Symbols**: Use `.globl` to make a label visible outside the current file (typically for the program entry point).
    ```assembly
    .globl entrypoint
    entrypoint:
        mov64 r0, 0
        exit
    ```

## Comments

Comments usually start with `//` or `#` and extend to the end of the line.

```assembly
// This is a single-line comment
mov64 r1, 42  # This is also a comment
```

## Instruction Format

Instructions generally follow the format: `mnemonic operand1, operand2, ...`
For most sBPF instructions, operands are typically ordered `destination, source` or `destination, immediate`.

```assembly
add64 r1, r2   // r1 = r1 + r2 (Register to Register)
mov64 r3, 100  // r3 = 100 (Immediate to Register)
```

## Operands

-   **Registers**: sBPF has 11 registers: `r0` through `r9` (general purpose), `r10` (frame pointer, aliased as `fp`), and `pc` (program counter, implicitly used). `r0` typically holds the return value or result. `r1` - `r5` are often used for arguments, `r6` - `r9` are callee-saved. In strict GAS syntax, registers are often prefixed with `%` (e.g., `%r0`), but this prefix is commonly omitted in sBPF assembly examples and documentation.
-   **Immediate Values**: Constant integer values. In GAS syntax, they are often prefixed with `$` (e.g., `$123`), but in sBPF assembly output/examples, the prefix is also often omitted (e.g., `123`).
    ```assembly
    mov64 r1, 42    // Move immediate 42 into r1
    add64 r1, -10   // Add immediate -10 to r1
    ```
-   **Memory Addressing**: Memory access uses a base register plus an offset: `offset(%reg)`. This signifies the memory location at address `register_value + offset`.
    ```assembly
    ldxw r1, 8(%fp)   // Load 4 bytes from address fp+8 into r1
    stb 0(%r2), r3    // Store 1 byte from r3 into address r2+0
    ```
    Note: `lddw` (load double word) uses a pseudo-instruction format involving an immediate value split across two instructions in the bytecode, but often represented abstractly in assembly.

This primer covers the basics. Refer to the opcode tables and instruction set details for specific instruction syntax and behavior.

---

| [← Up: Reference](./README.md) | [Next: Opcode Tables →](./02_opcode_tables.md) |
|:------------------------------:|:-----------------------------------------------:| 