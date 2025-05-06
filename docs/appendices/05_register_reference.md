# Register Set and Error Handling Reference

This appendix provides a reference for the sBPF register set, calling conventions, and error handling mechanisms.

## Register Set

sBPF provides 11 general-purpose 64-bit registers (R0-R10), with specific roles and preservation conventions:

| Register | Purpose | Preserved Across Calls | Notes |
|----------|---------|------------------------|-------|
| r0 | Return value/Scratch register | No | Contains the result or status code after function calls |
| r1 - r5 | Function arguments / Scratch | No | Used to pass first 5 parameters to functions |
| r6 - r9 | Callee-saved registers | Yes | Must contain the same value on function exit as on entry |
| r10 (fp) | Frame pointer | Yes (Read-only) | Points to the base (highest address) of the current stack frame. Read-only within program code. Automatically managed during `call`/`exit`. |
| pc | Program counter | N/A | Implicitly managed by the VM (not directly accessible) |

## Register Conventions

### Parameter Passing

- First 5 arguments are passed in registers r1-r5
- Additional arguments must be passed on the stack
- 64-bit values use a single register

### Return Values

- Return values are stored in r0
- Multiple return values must use the stack or a pointer to a structure

### Register Preservation

- Functions must preserve (save and restore) registers r6-r9 if they modify them
- Registers r0-r5 can be freely modified by any function

## Status Codes and Error Handling

sBPF programs terminate by executing the `exit` instruction. The value placed in register `r0` before `exit` serves as the status code:

| Status Code in r0 | Constant | Description |
|-------------------|----------|-------------|
| 0 | SUCCESS | Operation completed successfully |
| > 0 | Error Codes | Program-specific or standard error codes indicating failure |

## Common Runtime Errors

The following runtime errors may terminate a program's execution, often implicitly returning a specific error code:

| Error | Description | Prevention |
|-------|-------------|-----------|
| Access Violation | Memory access outside permitted regions or against permissions | Ensure all memory accesses (loads/stores) use valid addresses and offsets within the stack, heap, or input buffer bounds. |
| Invalid Instruction | Encountered an invalid or unsupported instruction opcode | Verify instruction compatibility with the target sBPF version (v1/v2). Ensure ELF is correctly formed. |
| Division by Zero | Division or modulo instruction attempted with a zero divisor | Add explicit checks for zero before division/modulo operations, especially when the divisor comes from a register. |
| Max Call Depth Exceeded | Exceeded the maximum allowed nested call depth (typically 64) | Limit recursion depth; prefer iteration when possible. |
| CPU Budget Exceeded | Program consumed more compute units than allocated for the transaction | Optimize code to reduce instruction count, avoid unnecessary loops or complex calculations. |
| Verifier Failure | Program failed static verification checks before execution | Ensure code adheres to sBPF rules (e.g., jump targets within bounds, valid register usage). |

## Stack Usage

- The stack grows downward (toward lower addresses).
- The frame pointer (`fp`/r10) points to the **base (highest address)** of the current stack frame.
- Stack accesses use negative offsets relative to `fp` (e.g., `ldxdw r0, -8(%fp)`).
- The stack should be kept 8-byte aligned.
- SBPFv2 allows dynamic stack allocation *within* a function using the `add_stk` instruction.

## Version Differences Summary

| Feature | SBPFv1 | SBPFv2 |
|---------|--------|--------|
| Stack Frame Handling | Default frame size (e.g., 4KiB or 8KiB) is typically allocated by `call`/`exit` instructions. The `ADD_STK` instruction is not available. | `call`/`exit` instructions manage basic frame linkage (saving return address, old FP). The new frame pointer is typically set based on the caller's stack pointer. The callee can then use `add_stk` for dynamic stack allocation within its frame. |
| Key Instructions | Has `neg32/64`, `le16/32/64`. Lacks `sdiv/srem`, `*hmul`, `hor64`, `lddw`, `add_stk`. | Lacks `neg`, `le`. Has `sdiv/srem`, `*hmul`, `hor64`, `lddw`, `add_stk`. |
| Register Use | Same 11 registers | Same 11 registers |
| Preservation Rules | Same | Same |
| Error Behavior | Terminates with status in r0 | Terminates with status in r0 |

---

| [← Previous: Compiler and Linker Flags](./04_build_flags.md) | [Up: Appendices](./README.md) |
|:------------------------------------------------------------:|:-----------------------------:| 