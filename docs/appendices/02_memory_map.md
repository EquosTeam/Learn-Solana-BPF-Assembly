# Memory Map Reference

This appendix describes the virtual memory map used by sBPF programs, detailing the memory regions, their purposes, and access permissions.

## Memory Regions

sBPF programs operate within a fixed virtual memory map with the following standard regions:

| Region           | Start Address              | Usable Size Limit | Reserved VA Size | Access     | Description |
|------------------|----------------------------|-------------------|------------------|------------|-------------|
| Program Code     | `0x100000000`              | ~32 KiB¹          | 32MB             | Read-only  | Program instructions & read-only data (.rodata) loaded by the VM |
| Stack            | `0x200000000`              | 32 KiB            | 32MB             | Read/Write | Stack memory, grows downward |
| Heap             | `0x300000000`              | Variable          | 32MB             | Read/Write | Dynamically allocated memory via syscalls |
| Input Parameters | `0x400000000`              | Variable          | 32MB             | Read-only  | Program input buffer, size depends on transaction |

*¹ Program size is limited by instruction count (max 4096 instructions = 32 KiB). Read-only data is typically placed within this segment by the linker.*
*Each region is assigned a large virtual address space (e.g., 32MB) for isolation, but the functional limits are much smaller.*

## Memory Access Rules

- Accessing memory outside permitted regions or exceeding usable limits results in an `AccessViolation` error.
- The stack pointer (`fp`, register R10) initially points **one byte past the end** of the usable stack region (e.g., `0x200008000` if stack is 32KiB).
- Memory accesses should be properly aligned (e.g., 8-byte accesses should be 8-byte aligned) to avoid performance penalties or potential errors.
- In SBPFv2, the `add_stk` instruction can be used to modify the stack pointer directly.

## Memory Layout Diagram (Conceptual)

```
+-----------------------+ <- VA End (Higher Addresses)
|                       |
|    Input Parameters   |
|      (Read-only)      |
|                       |
+-----------------------+ 0x400000000
|                       |
|         Heap          |
|      (Read/Write)     |
|      (Variable size)  |
+-----------------------+ 0x300000000
|                       |
|         Stack         |
|      (Read/Write)     |
|       (32 KiB)        |
+-----------------------+ 0x200000000
|                       |
|     Program Code      |
|  & Read-only Data     |
|      (~32 KiB)        |
+-----------------------+ 0x100000000
|                       |
|      Unused           |
|                       |
+-----------------------+ 0x000000000 <- VA Start
```
*(Note: Diagram shows conceptual layout; exact boundaries depend on usable sizes.)*

## Stack Frame Layout

When an internal function call occurs, the VM pushes a stack frame containing:

1. Caller-saved registers (R6-R9)
2. The calling function's frame pointer (FP)
3. The return address (PC + 1)

In SBPFv2, the VM implicitly reserves a fixed amount of stack space (e.g., 256 bytes) per call. In SBPFv1, stack frame management might involve explicit stack pointer adjustments.

---

| [← Previous: Opcode Reference](./01_opcode_reference.md) | [Up: Appendices](./README.md) | [Next: Sample Linker Script →](./03_linker_script.md) |
|:--------------------------------------------------------:|:-----------------------------:|:-------------------------------------------------------:| 