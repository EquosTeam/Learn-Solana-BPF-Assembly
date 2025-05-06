[Home](../../README.md) > [Foundations](./README.md) > Execution Model

---

# Execution Model

## Register Set

sBPF inherits the **11 general purpose 64-bit registers** from eBPF.  The
calling convention enforced by the Solana runtime is identical to Linux eBPF
with a few extra rules:

| Register | Role (Solana)                    | Callee-saved? |
|----------|----------------------------------|---------------|
| **r0**   | Return value                     | No            |
| **r1**   | Pointer to *input buffer*        | No            |
| **r2–r5**| Scratch / helper arguments       | No            |
| **r6–r9**| Free to use, **must** be saved   | Yes           |
| **r10**  | Frame pointer (stack base)       | Yes, read-only|

Registers `r6`–`r9` must contain the **same value** on `exit` as they did on
entry; this is required by the **calling convention** (handled implicitly by
function prologue/epilogue and the `call`/`exit` instructions).

> **Frame Pointer vs. Stack Pointer:** While `R10` is the read-only *frame*
> pointer typically pointing to the base of the current stack frame, stack
> management generally uses negative offsets from `R10` (e.g., `[r10 - 8]`).
> SBPF v2 introduces the `ADD_STK` instruction which allows modifying the VM's
> conceptual stack pointer for dynamic frames, but `R10` itself remains fixed
> within a function frame after the prologue.

> **Instruction Pointer:** The program counter (instruction pointer) is not
> directly accessible or modifiable by program code.

> **No Flags Register:** sBPF lacks a traditional CPU flags register (Zero,
> Carry, etc.). Conditional jumps perform comparisons directly.

> **32-bit subregisters** – Writing to a 32-bit variant of a register (e.g.
> `w0`, `w1`, …) **zeroes** the upper 32 bits automatically (e.g., `mov r0,
> 0xFFFFFFFF` followed by `add32 r0, 1` results in `r0 = 0x00000000`). This is
> important when mixing 32-bit and 64-bit ALU operations.

## Fixed Virtual Memory Map  *(Solana Docs – Programs FAQ)*

| Region | Start address  | Purpose                                          |
|--------|----------------|--------------------------------------------------|
| text   | `0x100000000`  | Code and read-only data                          |
| stack  | `0x200000000`  | Total 32 KiB stack region, grows **downwards**   |
| heap   | `0x300000000`  | Optional heap via `sol_alloc_free`               |
| input  | `0x400000000`  | Serialized program parameters                    |

Read-only data (like constants or strings defined in `.rodata`) typically resides
within the `.text` segment. Addresses of labels within this data are often
loaded into registers using the `lddw` pseudo-instruction.

Attempting to access unmapped addresses triggers an **AccessViolation** and the
transaction fails.

### Stack Usage

`r10` points to the **base of the current stack frame** (highest address).
Local variables are addressed with negative offsets relative to `r10`, for
example: `ldxdw r0, [r10 - 8]` accesses the first 8 bytes on the stack.

The stack is 32 KiB deep by default.

## Entrypoint & Exit

The entry symbol (often `e` or `entrypoint`) must be exported with `.globl` and
is selected as `ENTRY()` in the linker script.

On entry:

* `r1` = `0x4000_0000` (pointer to parameter buffer)
* All other registers are **zero**.

Return by placing a 64-bit value in `r0` and executing the `exit` instruction.
*   If the `call_depth` (see [Call Instructions](#instruction-encoding-basics)) is zero, the program terminates. The value in `r0` is the program's result code, where `0` indicates `SUCCESS`.
*   If `call_depth` is greater than zero, `exit` functions as a return from the current function. It restores caller-saved registers (`r6-r9`), the caller's frame pointer (`r10`), and the return address from the stack, then continues execution at the return address. The value in `r0` serves as the return value of the function call.

> **Note:** The runtime implicitly adds a prologue/epilogue to handle setup and
> teardown, including the initial register state and final return.

```asm
.globl e
 e:
     /* your code */
     exit            ; Return 0 (OK)
``` 

## Other VM Limits

Solana enforces several hard limits at the VM level:

* **`PROG_MAX_INSNS (4096)`** – the maximum number of 8-byte instructions a
  program may contain.  This mirrors the Linux eBPF limit and guarantees quick
  termination.
* **Division-by-Zero Checks:** Integer division/modulo instructions implicitly
  check for division by zero; a violation halts the program.
* **Call-depth limit (64)** – nested `call` instructions cannot exceed 64
  frames.
* **Stack frame size** – While sBPF supports dynamic stack frames (especially
  v1), the calling convention involves pushing a frame onto the stack. The
  default frame size used by `call`/`exit` in sBPF v1 is 4KiB or 8KiB, not a
  fixed small value. Refer to `call` instruction details for specifics.

## Instruction Encoding Basics

Every sBPF instruction is **64 bits** wide:

```
|  8-bit  |4|4| 16-bit | 32-bit |
| opcode |d|s| offset | immediate |
```

* **`opcode`** – Encodes the *class* (`BPF_LD`, `BPF_ALU64`, `BPF_JMP`, …), the
  *size* modifier (`BPF_DW`, `BPF_W`, `BPF_H`, `BPF_B`) and the *source* operand
  selector (`BPF_K` for immediates, `BPF_X` for register).
* **`d` / `s`** – 4-bit *destination* and *source* register indices (`0–10`).
* **`offset`** – 16-bit signed offset used by memory and branch instructions.
* **`immediate`** – 32-bit signed literal or helper/syscall identifier.

The `call` instruction has two main forms in sBPF:
*   **`CALL_IMM rs imm`**:
    *   If `rs` is **non-zero** (typically `r1`-`r5` for arguments): It's an
        **internal program call**. `imm` is a 32-bit key used to look up the
        target function's address in the program's function map.
    *   If `rs` is **zero** (`r0`): It's an **external call** (syscall) to the
        Solana runtime. `imm` identifies the specific syscall function.
*   **`CALL_REG rs imm`**: (Note: `imm` usage varies by SBPF version)
    *   In SBPF **v1**, `imm` (0-9) specifies the register (`r0`-`r9`) holding
        the target function address.
    *   In SBPF **v2**, `rs` specifies the register holding the target function
        address (and `imm` is often unused/zero).

`lddw` (load 64-bit immediate) is a **pseudo-instruction**, typically available in SBPFv2 and later (controlled by the VM's `enable_lddw()` flag). It loads a full 64-bit immediate value into a destination register and occupies *two* consecutive 64-bit instruction slots (16 bytes total):
*   The first instruction slot uses the `BPF_LD_IMM_DW` opcode (e.g., `0x18`), specifies the destination register `rd` in its `d` field, and its 32-bit `immediate` field holds the lower 32 bits of the 64-bit value. The `s` (source register) field in this first instruction is typically 0.
*   The second instruction slot *must* have its `opcode`, `d` (dst register), `s` (src register), and `offset` fields set to zero. Its 32-bit `immediate` field holds the upper 32 bits of the 64-bit value.
The program counter is advanced by 2 after an `lddw` instruction is executed.

## Endianness

The VM is **little-endian**.  Use the `be16`/`le16`, `be32`/`le32`,
`be64`/`le64` byte-swap instructions (`BPF_ALU | BPF_END`) when operating on
protocol data in big-endian order.

## Input Buffer Anatomy

When a Solana program is invoked, the runtime serializes necessary information into a single buffer located at a fixed virtual address: `0x400000000`. The program's entrypoint receives a pointer to the start of this buffer. The program (or libraries like `solana-program`) is responsible for deserializing this data.

The structure of this buffer, as defined by the latest BPF Loader (using little-endian encoding for multi-byte values), is as follows:

1.  **Number of Accounts** (`u64`, 8 bytes):
    *   The total count of account references passed to the instruction.

2.  **Account Information Array** (Variable size):
    *   An array containing serialized `AccountInfo`-like structures. For *each* account in the transaction's instruction:
        *   **Duplicate Marker** (`u8`, 1 byte): Indicates if this account entry is a duplicate of a preceding entry in the array.
            *   `0xff`: This entry represents a unique account (not previously listed).
            *   `0` to `N-1`: This entry is a duplicate of the account at the specified 0-based index within this array. Duplicates only contain this marker and padding.
        *   **If Duplicate:**
            *   **Padding** (7 bytes): Fills the remaining space for this entry.
        *   **If Not Duplicate:**
            *   **`is_signer`** (`u8` as bool, 1 byte): `1` if the account signed the transaction, `0` otherwise.
            *   **`is_writable`** (`u8` as bool, 1 byte): `1` if the program is allowed to modify the account's lamports or data, `0` otherwise.
            *   **`is_executable`** (`u8` as bool, 1 byte): `1` if the account holds a program (and is read-only), `0` otherwise.
            *   **Padding** (4 bytes): Aligns the following `Pubkey`.
            *   **Pubkey** (`[u8; 32]`, 32 bytes): The public key (address) of the account.
            *   **Owner** (`[u8; 32]`, 32 bytes): The public key of the program that owns this account.
            *   **Lamports** (`u64`, 8 bytes): The number of lamports held by the account. This value can be modified by the program if `is_writable` is true.
            *   **Data Length** (`u64`, 8 bytes): The size of the account's data field in bytes.
            *   **Account Data** (`[u8]`, `Data Length` bytes): The actual data stored in the account. This data can be modified by the program if `is_writable` is true.
            *   **Reallocation Padding** (10240 bytes / 10 KiB): Reserved memory immediately following the account data. This space allows the program to increase the account's data size using the `realloc` syscall, up to this limit.
            *   **Rent Epoch** (`u64`, 8 bytes): The epoch at which this account will next owe rent.
            *   **Alignment Padding** (0-7 bytes): Variable padding bytes (`0x00`) added to ensure the *next* account's entry (or the fields following the array if this is the last account) starts at an 8-byte aligned memory address relative to the beginning of the input buffer.

3.  **Instruction Data Length** (`u64`, 8 bytes):
    *   The number of bytes in the `Instruction Data` field that follows.

4.  **Instruction Data** (`[u8]`, `Instruction Data Length` bytes):
    *   The arbitrary byte array passed by the client in the transaction instruction. The program interprets this data according to its own logic (e.g., determining which internal function to call and its parameters).

5.  **Program ID** (`[u8; 32]`, 32 bytes):
    *   The public key (address) of the currently executing program.

This structured buffer provides the SBF program with all the necessary context (accounts, parameters, self-identification) to process an instruction.

## Next Steps

Now that you understand the execution model, continue to the [Toolchain Setup](./03_toolchain.md) to learn how to build your sBPF programs.

For more detailed information about the memory model, see the [Memory Map Reference](../appendices/02_memory_map.md).

---

| [← Previous: Introduction to sBPF](./01_introduction.md) | [Next: Toolchain Setup →](./03_toolchain.md) |
|:--------------------------------------------------------|---------------------------------------------:|
