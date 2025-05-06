# sBPF Instruction Set Details

This document provides details on the semantics, encoding, and execution environment of sBPF instructions, building upon the opcode tables.

## Memory Map

sBPF programs operate within a defined virtual address memory map. Accessing memory outside permitted regions results in an `AccessViolation` error. The typical layout is:

-   **Program Code (Read-only):** Starts at `0x100000000`
-   **Stack (Read/Write):** Starts at `0x200000000` (Grows downwards)
-   **Heap (Read/Write):** Starts at `0x300000000`
-   **Program Input Parameters (Read-only):** Starts at `0x400000000`

The exact sizes and permissions of these regions are determined when the program is loaded.

## Instruction Encoding

sBPF uses a fixed-size 64-bit instruction encoding format for most instructions. The v2 instruction `lddw` (Load Double Word) is an exception, occupying 128 bits (two 64-bit slots).

The standard 64-bit format is:

| Bits   | 63-56 (8) | 55-52 (4) | 51-48 (4) | 47-32 (16) | 31-0 (32)  |
| :----- | :-------- | :-------- | :-------- | :--------- | :--------- |
| Field  | `opcode`  | `dst`     | `src`     | `offset`   | `immediate`|
| Meaning| Operation | Dest Reg  | Source Reg| Addr/Jump Offset | Constant Value |

-   **`opcode` (8 bits)**: Defines the instruction class, operation mode, and specific action. See Opcode Field Structure below.
-   **`dst` (4 bits)**: Destination register index (0-9 for R0-R9). Can be R10 (`fp`) for store instructions (`st*`).
-   **`src` (4 bits)**: Source register index (0-9 for R0-R9, or 10 for R10/`fp` if used as a source). Used when the instruction takes a register source.
-   **`offset` (16 bits)**: Signed offset used primarily for memory addressing (`base_reg + offset`) and conditional/unconditional jumps (`PC + offset + 1`).
-   **`immediate` (32 bits)**: Signed immediate value used when the instruction takes a constant operand.

### Opcode Field Structure (Conceptual)

The 8-bit `opcode` field is typically composed of several sub-fields combined using bitwise OR:

-   **Instruction Class (Bits 7-5):** Defines the general category (e.g., Load, Store, ALU, Jump). Examples from `solana_sbpf::ebpf`:
    -   `BPF_LDX` (Load from Register offset - uses `BPF_MEM` mode)
    -   `BPF_STX` (Store Register - uses `BPF_MEM` mode)
    -   `BPF_ST` (Store Immediate - uses `BPF_MEM` mode)
    -   `BPF_ALU64_STORE` (Class for 64-bit ALU operations where results update the full 64-bit destination register)
    -   `BPF_ALU32_LOAD` (Class for 32-bit ALU operations where results update the lower 32-bits of the destination register, and the upper 32-bits are zeroed)
    -   `BPF_JMP` (Jump operations)
    -   `BPF_PQR` (Product/Quotient/Remainder - v2)
-   **Size Modifier (Bits 4-3):** Specifies the operand size. Examples:
    -   `BPF_W` (Word - 4 bytes)
    -   `BPF_H` (Half-word - 2 bytes)
    -   `BPF_B` (Byte - 1 byte)
    -   `BPF_DW` (Double word - 8 bytes)
-   **Mode Modifier / Source (Bits 2-0):** Indicates the source type or operation mode. Examples:
    -   `BPF_K` (Immediate value)
    -   `BPF_X` (Register source)
    -   `BPF_IMM` (Immediate mode - also used for specific opcodes like `CALL_IMM`)
    -   `BPF_MEM` (Memory access mode)
    -   For ALU/Jump classes, these bits often specify the exact operation (e.g., `BPF_ADD`, `BPF_JEQ`, `BPF_JSGT`).

The exact composition determines the specific instruction. For example, `add64_reg` combines elements indicating ALU64 class, register source (`BPF_X`), and the ADD operation. `ldxw` combines elements indicating a load class, word size (`BPF_W`), and memory mode (`BPF_MEM`).

### `lddw` Encoding (v2)

The `lddw` instruction loads a full 64-bit immediate value into a destination register. It spans two 64-bit instruction slots:

-   **Slot 1**: Contains the `lddw` opcode (`BPF_LD | BPF_DW | BPF_IMM`), `dst` register, and the *lower* 32 bits of the immediate value in the `immediate` field. The `src` field is 0.
-   **Slot 2**: Contains the *higher* 32 bits of the immediate value in its `immediate` field. All other fields (`opcode`, `dst`, `src`, `offset`) must be zero.

The Program Counter (PC) advances by 2 after executing an `lddw`.

## Instruction Categories

### ALU Instructions

Perform arithmetic (add, sub, mul, div, mod, neg), logical (or, and, xor), shift (lsh, rsh, arsh), and move (mov) operations.

-   **Width**: Instructions exist in 32-bit and 64-bit variants.
    -   32-bit operations typically operate on the lower 32 bits of the source operands, produce a 32-bit result, and then sign-extend this result to 64 bits before storing it in the destination register.
    -   64-bit operations use the full 64-bit width of the registers.
-   **Signedness**:
    -   In sBPF (unlike classic eBPF), `add32`, `sub32`, `mul32` perform *signed* arithmetic.
    -   Division (`div`) and modulo (`mod`) operations use unsigned semantics (`MDM`/`UDIV`/`UREM` variant) in all versions. Signed variants (`PQR` - `sdiv`, `srem`) are available only in v2. Division by zero results in an error (for immediate 0) or an execution flag (for register containing 0).
    -   Arithmetic shifts (`arsh`) perform sign-preserving right shifts. Logical shifts (`lsh`, `rsh`) treat the operand as unsigned.
    -   Comparisons in Jump instructions have both signed (`jsgt`, `jsge`, `jslt`, `jsle`) and unsigned (`jgt`, `jge`, `jlt`, `jle`) variants.
-   **Operands**: Can be register-register (`op rd, rs`) or register-immediate (`op rd, imm`).
-   **Shifts**: Shift amounts from registers are masked (`& 31` for 32-bit, `& 63` for 64-bit) to prevent excessive shifts. Immediate shifts must be within range [0, 31] or [0, 63] respectively (checked by verifier).
-   **v1 vs v2**:
    -   `neg32`/`neg64` are only available in v1.
    -   Signed division/modulo (`sdiv`/`srem`) are only available in v2.
    -   128-bit multiplication (`uhmul`/`shmul` - storing the high 64 bits) are only available in v2.
    -   `hor64` (bitwise OR with immediate shifted left by 32) is only available in v2.

### Jump Instructions

Control the program flow.

-   **Unconditional**: `ja offset` always jumps.
-   **Conditional**: `jxx rd, sop, offset` jumps if the condition between `rd` and `sop` (source register or immediate) is true.
-   **Target Calculation**: The jump target address is calculated as `PC + offset + 1`, where `PC` is the address of the *current* jump instruction, and `offset` is the 16-bit signed offset from the instruction encoding. This means the offset is relative to the *next* instruction's address.
-   **Conditions**: Include equality (`jeq`, `jne`), unsigned comparisons (`jgt`, `jge`, `jlt`, `jle`), signed comparisons (`jsgt`, `jsge`, `jslt`, `jsle`), and bitwise tests (`jset`).

### Load/Store Instructions

Interact with memory (stack, heap, input parameters).

-   **Addressing Mode**: `[base_reg + offset]`. The address is computed by adding the 16-bit signed `offset` to the value in the base register (`dst` for stores, `src` for loads). `fp` (`r10`) is commonly used as the base for stack access.
-   **Width**: Instructions exist for loading/storing 1 byte (`ldxb`/`stb`), 2 bytes (`ldxh`/`sth`), 4 bytes (`ldxw`/`stw`), and 8 bytes (`ldxdw`/`stdw`).
    -   Loads zero-extend the value to 64 bits after reading from memory (e.g., `ldxb` loads a byte and places it in the lowest 8 bits of `dst`, clearing the upper 56 bits).
    -   Stores write the lower N bytes of the source register/immediate to memory.
-   **Operands**:
    -   Loads (`ldx*`): `ldx<size> dst, [src+off]` (Load from address `src+off` into `dst`).
    -   Stores (`st*`): `st<size> [dst+off], src` (Store register `src` to address `dst+off`) or `st<size> [dst+off], imm` (Store immediate `imm` to address `dst+off`).
-   **Alignment**: Accesses should ideally be aligned to the size of the access (e.g., an `ldxdw` address should be 8-byte aligned). Misaligned accesses might be handled by the runtime but incur a performance penalty or potentially cause errors depending on the execution environment.
-   **`lddw` (v2)**: Special load instruction to load a full 64-bit immediate value. Uses two instruction slots. See Encoding section.

### Byte Swap Instructions

Change the endianness of the value in a register. Useful for ensuring consistent data format across different systems or network protocols.

-   `le<size> dst`: Convert `dst` to Little Endian format (v1 only). If the host is already little-endian (like x86-64), this might be a no-op conceptually, but the instruction exists.
-   `be<size> dst`: Convert `dst` to Big Endian format (All versions).
-   Size (`<size>`) can be 16, 32, or 64 bits.

### Call/Exit Instructions

Manage function calls (internal and external/syscalls) and program termination.

-   **`call_imm imm`**:
    -   Identified by the `BPF_CALL | BPF_IMM` opcode components. The specific behavior (internal call vs. syscall) depends on convention (often `src` register field being 0 for syscalls, non-zero for internal).
    -   *Internal Call*: Jumps to the function address associated with the pseudo-immediate `imm` (often representing a function ID resolved by the loader/runtime). Pushes a return frame onto the stack.
    -   *External Call (Syscall)*: Invokes a host system call identified by `imm`. Arguments are typically passed in registers `r1`-`r5`, and the result is returned in `r0`. The exact mechanism is host-defined.
    -   *(SBPFv3 Note)*: Later sBPF versions might use a dedicated `SYSCALL` opcode (`BPF_JMP | BPF_SYSCALL`).
-   **`call_reg rs`**: (Behavior differs by version)
    -   Identified by `BPF_CALL | BPF_X`.
    -   **v1**: Calls function index specified by the `immediate` field value (0-9). The target address seems to be implicitly derived (e.g., `R[imm]`?). This usage is less common/clear.
    -   **v2**: Calls function at the address contained in the source register `rs`. Pushes a return frame.
-   **Stack Frame**: When an internal call occurs, a frame is pushed onto the stack. Based on `sbpf_full.md`, this frame saves:
    -   Caller-saved registers: `r6`, `r7`, `r8`, `r9`.
    -   The caller's frame pointer (`fp`).
    -   The return address (`PC + 1`).
    *Caveat*: The precise order and byte layout of these elements on the stack are not specified in the provided documents and may depend on implementation details. The frame pointer (`fp`, R10) is updated to point to the base of the new frame.
-   **`exit`**:
    -   Identified by `BPF_JMP | BPF_EXIT`.
    -   If the call depth is > 0 (i.e., inside an internal function call), pops the most recent return frame from the stack, restoring the saved `r6`-`r9`, `fp`, and `PC`. Execution resumes in the caller at the instruction after the `call`. The value in `r0` serves as the return value.
    -   If the call depth is 0 (i.e., in the main program body), terminates the program execution. The value in `r0` is the final result of the program.
    -   *(SBPFv3 Note)*: Later sBPF versions might use a dedicated `RETURN` opcode (`BPF_JMP | BPF_RETURN`).

### Other Instructions (v2 Specific)

-   **`add_stk imm`**: Modifies the VM's internal stack pointer by the immediate value `imm`. This allows for dynamic stack allocation/deallocation within a function. The `r10` frame pointer register itself remains unchanged by this instruction. This instruction is available in sBPFv2 and is typically identified by the ALU ADD encoding (`BPF_ALU64 | BPF_ADD | BPF_K` or similar, depending on the exact `BPF_ALU64_STORE` definition) when the `dst` register field in the instruction is 0 and the relevant SBPFv2 feature flag is enabled.

---

| [← Previous: Opcode Tables](./02_opcode_tables.md) | [Up: Reference](./README.md) |
|:--------------------------------------------------:|:----------------------------:| 