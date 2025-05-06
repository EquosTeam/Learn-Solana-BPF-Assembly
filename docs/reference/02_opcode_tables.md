# sBPF Opcode Tables

This document provides a reference for sBPF instructions, categorized by function. The mnemonics listed generally follow the conventions used in Solana's internal tooling and documentation.

**Note on Hex Opcodes**: This table lists the corresponding constant names from the `solana_sbpf::ebpf` Rust module. The precise 8-bit hexadecimal values for these opcodes are not detailed in the provided reference documents and would require consulting the `solana-rbpf` source code or a definitive ISA specification.

**Note on Versions**: Version indicates if an instruction is common to both v1 and v2 ('All'), specific to v1 ('v1'), specific to v2 ('v2'), or potentially part of a later version like v3 ('v3'). Some behaviors differ between versions (e.g., 32-bit ALU signedness, division semantics).

## ALU Instructions (Arithmetic Logic Unit)

These perform arithmetic and logical operations. 32-bit operations generally operate on the lower 32 bits and sign-extend results to 64 bits. 64-bit operations use the full register width.

**Important**: In sBPF (unlike classic eBPF), 32-bit `add`, `sub`, and `mul` have *signed* semantics. Division/Modulo operations use unsigned semantics (`MDM` variant) in all versions, with additional signed variants (`PQR` variant) introduced in v2.

| Mnemonic     | Operands | Description                       | Version | Corresponding Constant (solana_sbpf::ebpf) |
| :----------- | :------- | :-------------------------------- | :------ | :----------------------------------------- |
| `add32_reg`  | `dst, src` | `dst = (i32)dst + (i32)src`       | All     | `ADD32_REG`                                |
| `add32_imm`  | `dst, imm` | `dst = (i32)dst + (i32)imm`       | All     | `ADD32_IMM`                                |
| `sub32_reg`  | `dst, src` | `dst = (i32)dst - (i32)src`       | All     | `SUB32_REG`                                |
| `sub32_imm`  | `dst, imm` | `dst = (i32)dst - (i32)imm`       | All     | `SUB32_IMM`                                |
| `mul32_reg`  | `dst, src` | `dst = (i32)dst * (i32)src`       | All     | `MUL32_REG`                                |
| `mul32_imm`  | `dst, imm` | `dst = (i32)dst * (i32)imm`       | All     | `MUL32_IMM`                                |
| `div32_reg`  | `dst, src` | `dst = (u32)dst / (u32)src`       | All     | `DIV32_REG` (*MDM/UDIV variant*)           |
| `div32_imm`  | `dst, imm` | `dst = (u32)dst / (u32)imm`       | All     | `DIV32_IMM` (*MDM/UDIV variant*)           |
| `mod32_reg`  | `dst, src` | `dst = (u32)dst % (u32)src`       | All     | `MOD32_REG` (*MDM/UREM variant*)           |
| `mod32_imm`  | `dst, imm` | `dst = (u32)dst % (u32)imm`       | All     | `MOD32_IMM` (*MDM/UREM variant*)           |
| `or32_reg`   | `dst, src` | `dst = (u32)dst \| (u32)src`      | All     | `OR32_REG`                                 |
| `or32_imm`   | `dst, imm` | `dst = (u32)dst \| (u32)imm`      | All     | `OR32_IMM`                                 |
| `and32_reg`  | `dst, src` | `dst = (u32)dst & (u32)src`       | All     | `AND32_REG`                                |
| `and32_imm`  | `dst, imm` | `dst = (u32)dst & (u32)imm`       | All     | `AND32_IMM`                                |
| `lsh32_reg`  | `dst, src` | `dst = (u32)dst << (src & 31)`    | All     | `LSH32_REG`                                |
| `lsh32_imm`  | `dst, imm` | `dst = (u32)dst << (imm & 31)`    | All     | `LSH32_IMM`                                |
| `rsh32_reg`  | `dst, src` | `dst = (u32)dst >> (src & 31)`    | All     | `RSH32_REG`                                |
| `rsh32_imm`  | `dst, imm` | `dst = (u32)dst >> (imm & 31)`    | All     | `RSH32_IMM`                                |
| `neg32`      | `dst`      | `dst = -(i32)dst`                 | v1      | `NEG32`                                    |
| `xor32_reg`  | `dst, src` | `dst = (u32)dst ^ (u32)src`       | All     | `XOR32_REG`                                |
| `xor32_imm`  | `dst, imm` | `dst = (u32)dst ^ (u32)imm`       | All     | `XOR32_IMM`                                |
| `mov32_reg`  | `dst, src` | `dst = (u32)src`                  | All     | `MOV32_REG`                                |
| `mov32_imm`  | `dst, imm` | `dst = (u32)imm`                  | All     | `MOV32_IMM`                                |
| `arsh32_reg` | `dst, src` | `dst = (i32)dst >> (src & 31)`    | All     | `ARSH32_REG`                               |
| `arsh32_imm` | `dst, imm` | `dst = (i32)dst >> (imm & 31)`    | All     | `ARSH32_IMM`                               |
| `add64_reg`  | `dst, src` | `dst = dst + src`                 | All     | `ADD64_REG`                                |
| `add64_imm`  | `dst, imm` | `dst = dst + imm`                 | All     | `ADD64_IMM`                                |
| `sub64_reg`  | `dst, src` | `dst = dst - src`                 | All     | `SUB64_REG`                                |
| `sub64_imm`  | `dst, imm` | `dst = dst - imm`                 | All     | `SUB64_IMM`                                |
| `mul64_reg`  | `dst, src` | `dst = dst * src`                 | All     | `MUL64_REG`                                |
| `mul64_imm`  | `dst, imm` | `dst = dst * imm`                 | All     | `MUL64_IMM`                                |
| `div64_reg`  | `dst, src` | `dst = (u64)dst / (u64)src`       | All     | `DIV64_REG` (*MDM/UDIV variant*)           |
| `div64_imm`  | `dst, imm` | `dst = (u64)dst / (u64)imm`       | All     | `DIV64_IMM` (*MDM/UDIV variant*)           |
| `mod64_reg`  | `dst, src` | `dst = (u64)dst % (u64)src`       | All     | `MOD64_REG` (*MDM/UREM variant*)           |
| `mod64_imm`  | `dst, imm` | `dst = (u64)dst % (u64)imm`       | All     | `MOD64_IMM` (*MDM/UREM variant*)           |
| `or64_reg`   | `dst, src` | `dst = dst \| src`                | All     | `OR64_REG`                                 |
| `or64_imm`   | `dst, imm` | `dst = dst \| imm`                | All     | `OR64_IMM`                                 |
| `and64_reg`  | `dst, src` | `dst = dst & src`                 | All     | `AND64_REG`                                |
| `and64_imm`  | `dst, imm` | `dst = dst & imm`                 | All     | `AND64_IMM`                                |
| `lsh64_reg`  | `dst, src` | `dst = dst << (src & 63)`         | All     | `LSH64_REG`                                |
| `lsh64_imm`  | `dst, imm` | `dst = dst << (imm & 63)`         | All     | `LSH64_IMM`                                |
| `rsh64_reg`  | `dst, src` | `dst = (u64)dst >> (src & 63)`    | All     | `RSH64_REG`                                |
| `rsh64_imm`  | `dst, imm` | `dst = (u64)dst >> (imm & 63)`    | All     | `RSH64_IMM`                                |
| `neg64`      | `dst`      | `dst = -dst`                      | v1      | `NEG64`                                    |
| `xor64_reg`  | `dst, src` | `dst = dst ^ src`                 | All     | `XOR64_REG`                                |
| `xor64_imm`  | `dst, imm` | `dst = dst ^ imm`                 | All     | `XOR64_IMM`                                |
| `mov64_reg`  | `dst, src` | `dst = src`                       | All     | `MOV64_REG`                                |
| `mov64_imm`  | `dst, imm` | `dst = imm`                       | All     | `MOV64_IMM`                                |
| `arsh64_reg` | `dst, src` | `dst = (i64)dst >> (src & 63)`    | All     | `ARSH64_REG`                               |
| `arsh64_imm` | `dst, imm` | `dst = (i64)dst >> (imm & 63)`    | All     | `ARSH64_IMM`                               |
| `hor64`      | `dst, imm` | `dst = dst \| (imm << 32)`        | v2      | `HOR64_IMM`                                |
| `sdiv32_reg` | `dst, src` | `dst = (i32)dst / (i32)src`       | v2      | `SDIV32_REG` (*PQR variant*)               |
| `sdiv32_imm` | `dst, imm` | `dst = (i32)dst / (i32)imm`       | v2      | `SDIV32_IMM` (*PQR variant*)               |
| `srem32_reg` | `dst, src` | `dst = (i32)dst % (i32)src`       | v2      | `SREM32_REG` (*PQR variant*)               |
| `srem32_imm` | `dst, imm` | `dst = (i32)dst % (i32)imm`       | v2      | `SREM32_IMM` (*PQR variant*)               |
| `sdiv64_reg` | `dst, src` | `dst = (i64)dst / (i64)src`       | v2      | `SDIV64_REG` (*PQR variant*)               |
| `sdiv64_imm` | `dst, imm` | `dst = (i64)dst / (i64)imm`       | v2      | `SDIV64_IMM` (*PQR variant*)               |
| `srem64_reg` | `dst, src` | `dst = (i64)dst % (i64)src`       | v2      | `SREM64_REG` (*PQR variant*)               |
| `srem64_imm` | `dst, imm` | `dst = (i64)dst % (i64)imm`       | v2      | `SREM64_IMM` (*PQR variant*)               |
| `uhmul64_reg`| `dst, src` | `dst = ((u128)dst * (u128)src)>>64`| v2      | `UHMUL64_REG`                              |
| `uhmul64_imm`| `dst, imm` | `dst = ((u128)dst * (u128)imm)>>64`| v2      | `UHMUL64_IMM`                              |
| `shmul64_reg`| `dst, src` | `dst = ((i128)dst * (i128)src)>>64`| v2      | `SHMUL64_REG`                              |
| `shmul64_imm`| `dst, imm` | `dst = ((i128)dst * (i128)imm)>>64`| v2      | `SHMUL64_IMM`                              |

## Jump Instructions

These alter the program flow based on conditions or unconditionally. `offset` is a 16-bit signed value relative to the *next* instruction (`PC + 1`). Target address = `PC + offset + 1`.

| Mnemonic   | Operands     | Condition                    | Version | Corresponding Constant (solana_sbpf::ebpf) |
| :--------- | :----------- | :--------------------------- | :------ | :----------------------------------------- |
| `ja`       | `offset`     | Jump always                  | All     | `JA`                                       |
| `jeq_reg`  | `dst, src, offset` | `if dst == src`          | All     | `JEQ_REG`                                  |
| `jeq_imm`  | `dst, imm, offset` | `if dst == imm`          | All     | `JEQ_IMM`                                  |
| `jgt_reg`  | `dst, src, offset` | `if (u64)dst > (u64)src` | All     | `JGT_REG`                                  |
| `jgt_imm`  | `dst, imm, offset` | `if (u64)dst > imm`      | All     | `JGT_IMM`                                  |
| `jge_reg`  | `dst, src, offset` | `if (u64)dst >= (u64)src`| All     | `JGE_REG`                                  |
| `jge_imm`  | `dst, imm, offset` | `if (u64)dst >= imm`     | All     | `JGE_IMM`                                  |
| `jlt_reg`  | `dst, src, offset` | `if (u64)dst < (u64)src` | All     | `JLT_REG`                                  |
| `jlt_imm`  | `dst, imm, offset` | `if (u64)dst < imm`      | All     | `JLT_IMM`                                  |
| `jle_reg`  | `dst, src, offset` | `if (u64)dst <= (u64)src`| All     | `JLE_REG`                                  |
| `jle_imm`  | `dst, imm, offset` | `if (u64)dst <= imm`     | All     | `JLE_IMM`                                  |
| `jset_reg` | `dst, src, offset` | `if dst & src`           | All     | `JSET_REG`                                 |
| `jset_imm` | `dst, imm, offset` | `if dst & imm`           | All     | `JSET_IMM`                                 |
| `jne_reg`  | `dst, src, offset` | `if dst != src`          | All     | `JNE_REG`                                  |
| `jne_imm`  | `dst, imm, offset` | `if dst != imm`          | All     | `JNE_IMM`                                  |
| `jsgt_reg` | `dst, src, offset` | `if (i64)dst > (i64)src` | All     | `JSGT_REG`                                 |
| `jsgt_imm` | `dst, imm, offset` | `if (i64)dst > imm`      | All     | `JSGT_IMM`                                 |
| `jsge_reg` | `dst, src, offset` | `if (i64)dst >= (i64)src`| All     | `JSGE_REG`                                 |
| `jsge_imm` | `dst, imm, offset` | `if (i64)dst >= imm`     | All     | `JSGE_IMM`                                 |
| `jslt_reg` | `dst, src, offset` | `if (i64)dst < (i64)src` | All     | `JSLT_REG`                                 |
| `jslt_imm` | `dst, imm, offset` | `if (i64)dst < imm`      | All     | `JSLT_IMM`                                 |
| `jsle_reg` | `dst, src, offset` | `if (i64)dst <= (i64)src`| All     | `JSLE_REG`                                 |
| `jsle_imm` | `dst, imm, offset` | `if (i64)dst <= imm`     | All     | `JSLE_IMM`                                 |

## Memory Instructions

These load data from memory into registers or store data from registers into memory. Memory addresses are calculated as `base_reg + offset`. `offset` is a 16-bit signed value.

| Mnemonic  | Operands        | Description                               | Version | Corresponding Constant (solana_sbpf::ebpf) |
| :-------- | :-------------- | :---------------------------------------- | :------ | :----------------------------------------- |
| `ldxb`    | `dst, [src+off]`| `dst = (u8)*(src+off)`                    | All     | `LD_B_REG` / `LD_1B_REG`                   |
| `ldxh`    | `dst, [src+off]`| `dst = (u16)*(src+off)`                   | All     | `LD_H_REG` / `LD_2B_REG`                   |
| `ldxw`    | `dst, [src+off]`| `dst = (u32)*(src+off)`                   | All     | `LD_W_REG` / `LD_4B_REG`                   |
| `ldxdw`   | `dst, [src+off]`| `dst = (u64)*(src+off)`                   | All     | `LD_DW_REG` / `LD_8B_REG`                  |
| `stb_reg` | `[dst+off], src`| `*(u8*)(dst+off) = (u8)src`               | All     | `ST_B_REG` / `ST_1B_REG`                   |
| `stb_imm` | `[dst+off], imm`| `*(u8*)(dst+off) = (u8)imm`               | All     | `ST_B_IMM` / `ST_1B_IMM`                   |
| `sth_reg` | `[dst+off], src`| `*(u16*)(dst+off) = (u16)src`              | All     | `ST_H_REG` / `ST_2B_REG`                   |
| `sth_imm` | `[dst+off], imm`| `*(u16*)(dst+off) = (u16)imm`              | All     | `ST_H_IMM` / `ST_2B_IMM`                   |
| `stw_reg` | `[dst+off], src`| `*(u32*)(dst+off) = (u32)src`              | All     | `ST_W_REG` / `ST_4B_REG`                   |
| `stw_imm` | `[dst+off], imm`| `*(u32*)(dst+off) = (u32)imm`              | All     | `ST_W_IMM` / `ST_4B_IMM`                   |
| `stdw_reg`| `[dst+off], src`| `*(u64*)(dst+off) = (u64)src`              | All     | `ST_DW_REG` / `ST_8B_REG`                  |
| `stdw_imm`| `[dst+off], imm`| `*(u64*)(dst+off) = (u64)imm`              | All     | `ST_DW_IMM` / `ST_8B_IMM`                  |
| `lddw`    | `dst, imm64`    | `dst = imm64` (Uses two instruction slots)| v2      | `LD_DW_IMM`                                |

## Byte Swap Instructions

These change the byte order (endianness) of data within a register. Assumes the host is Little Endian.

| Mnemonic | Operands | Description                             | Version | Corresponding Constant (solana_sbpf::ebpf) |
| :------- | :------- | :-------------------------------------- | :------ | :----------------------------------------- |
| `le16`   | `dst`    | `dst = htole16(dst)` (Host to Little End) | v1      | `LE` (with imm=16)                         |
| `le32`   | `dst`    | `dst = htole32(dst)`                    | v1      | `LE` (with imm=32)                         |
| `le64`   | `dst`    | `dst = htole64(dst)`                    | v1      | `LE` (with imm=64)                         |
| `be16`   | `dst`    | `dst = htobe16(dst)` (Host to Big End)    | All     | `BE` (with imm=16)                         |
| `be32`   | `dst`    | `dst = htobe32(dst)`                    | All     | `BE` (with imm=32)                         |
| `be64`   | `dst`    | `dst = htobe64(dst)`                    | All     | `BE` (with imm=64)                         |

## Call/Exit Instructions

| Mnemonic  | Operands | Description                                             | Version | Corresponding Constant (solana_sbpf::ebpf) |
| :-------- | :------- | :------------------------------------------------------ | :------ | :----------------------------------------- |
| `call_reg`| `rs`     | Function call to address in `rs` (v2) or `R[imm]` (v1)  | All     | `CALL_REG`                                 |
| `call_imm`| `imm`    | Function call to immediate `imm` (Internal/Syscall)     | All     | `CALL_IMM`                                 |
| `exit`    |          | Return from function or terminate program               | All     | `EXIT`                                     |
| `syscall` | `imm`    | Syscall identified by `imm`                             | v3?     | `SYSCALL`                                  |
| `return`  |          | Return `r0`                                             | v3?     | `RETURN`                                   |

*Note*: `SYSCALL` and `RETURN` are listed in `solana_sbpf::ebpf` potentially for sBPFv3, but the primary documentation focuses on v1/v2 where `call_imm` handles syscalls and `exit` handles returns.

## Other Instructions

| Mnemonic | Operands | Description                              | Version | Corresponding Constant (solana_sbpf::ebpf) |
| :------- | :------- | :--------------------------------------- | :------ | :----------------------------------------- |
| `add_stk`| `imm`    | Modifies the VM's stack pointer by `imm`. Affects subsequent stack accesses. | v2      | `ADD64_IMM` (when dst_reg=0, SBPFv2+)    |

---

| [← Previous: GAS Syntax Primer](./01_gas_syntax.md) | [Up: Reference](./README.md) | [Next: Instruction Set →](./03_instruction_set.md) |
|:---------------------------------------------------:|:----------------------------:|:-------------------------------------------------:|
