# Complete Opcode Reference Table

This appendix provides a comprehensive reference of all sBPF opcodes, organized by category with their binary encoding components.

## ALU Instructions (64-bit)

| Mnemonic | Description | Opcode Hex | Class | Mode/Source | Version |
|----------|-------------|------------|-------|-------------|---------|
| `add64_imm` | `dst += imm` | `0x07` | `BPF_ALU64_STORE` | `BPF_ADD \| BPF_K` | All |
| `add64_reg` | `dst += src` | `0x0f` | `BPF_ALU64_STORE` | `BPF_ADD \| BPF_X` | All |
| `sub64_imm` | `dst -= imm` | `0x17` | `BPF_ALU64_STORE` | `BPF_SUB \| BPF_K` | All |
| `sub64_reg` | `dst -= src` | `0x1f` | `BPF_ALU64_STORE` | `BPF_SUB \| BPF_X` | All |
| `mul64_imm` | `dst *= imm` | `0x27` | `BPF_ALU64_STORE` | `BPF_MUL \| BPF_K` | All |
| `mul64_reg` | `dst *= src` | `0x2f` | `BPF_ALU64_STORE` | `BPF_MUL \| BPF_X` | All |
| `div64_imm` | `dst /= imm (unsigned)` | `0x37` | `BPF_ALU64_STORE` | `BPF_DIV \| BPF_K` | All |
| `div64_reg` | `dst /= src (unsigned)` | `0x3f` | `BPF_ALU64_STORE` | `BPF_DIV \| BPF_X` | All |
| `or64_imm` | `dst \|= imm` | `0x47` | `BPF_ALU64_STORE` | `BPF_OR \| BPF_K` | All |
| `or64_reg` | `dst \|= src` | `0x4f` | `BPF_ALU64_STORE` | `BPF_OR \| BPF_X` | All |
| `and64_imm` | `dst &= imm` | `0x57` | `BPF_ALU64_STORE` | `BPF_AND \| BPF_K` | All |
| `and64_reg` | `dst &= src` | `0x5f` | `BPF_ALU64_STORE` | `BPF_AND \| BPF_X` | All |
| `lsh64_imm` | `dst <<= imm` | `0x67` | `BPF_ALU64_STORE` | `BPF_LSH \| BPF_K` | All |
| `lsh64_reg` | `dst <<= src` | `0x6f` | `BPF_ALU64_STORE` | `BPF_LSH \| BPF_X` | All |
| `rsh64_imm` | `dst >>= imm (logical)` | `0x77` | `BPF_ALU64_STORE` | `BPF_RSH \| BPF_K` | All |
| `rsh64_reg` | `dst >>= src (logical)` | `0x7f` | `BPF_ALU64_STORE` | `BPF_RSH \| BPF_X` | All |
| `neg64` | `dst = -dst` | `0x87` | `BPF_ALU64_STORE` | `BPF_NEG` | v1 |
| `mod64_imm` | `dst %= imm (unsigned)` | `0x97` | `BPF_ALU64_STORE` | `BPF_MOD \| BPF_K` | All |
| `mod64_reg` | `dst %= src (unsigned)` | `0x9f` | `BPF_ALU64_STORE` | `BPF_MOD \| BPF_X` | All |
| `xor64_imm` | `dst ^= imm` | `0xa7` | `BPF_ALU64_STORE` | `BPF_XOR \| BPF_K` | All |
| `xor64_reg` | `dst ^= src` | `0xaf` | `BPF_ALU64_STORE` | `BPF_XOR \| BPF_X` | All |
| `mov64_imm` | `dst = imm` | `0xb7` | `BPF_ALU64_STORE` | `BPF_MOV \| BPF_K` | All |
| `mov64_reg` | `dst = src` | `0xbf` | `BPF_ALU64_STORE` | `BPF_MOV \| BPF_X` | All |
| `arsh64_imm` | `dst >>= imm (arithmetic)` | `0xc7` | `BPF_ALU64_STORE` | `BPF_ARSH \| BPF_K` | All |
| `arsh64_reg` | `dst >>= src (arithmetic)` | `0xcf` | `BPF_ALU64_STORE` | `BPF_ARSH \| BPF_X` | All |
| `sdiv64_imm` | `dst /= imm (signed)` | `0x17` | `BPF_PQR` | `BPF_SDIV \| BPF_K` | v2 |
| `sdiv64_reg` | `dst /= src (signed)` | `0x1f` | `BPF_PQR` | `BPF_SDIV \| BPF_X` | v2 |
| `srem64_imm` | `dst %= imm (signed)` | `0x27` | `BPF_PQR` | `BPF_SREM \| BPF_K` | v2 |
| `srem64_reg` | `dst %= src (signed)` | `0x2f` | `BPF_PQR` | `BPF_SREM \| BPF_X` | v2 |
| `uhmul64_imm` | `dst = ((u128)dst * (u128)imm)>>64` | `0x77` | `BPF_PQR` | `BPF_UHMUL \| BPF_K` | v2 |
| `uhmul64_reg` | `dst = ((u128)dst * (u128)src)>>64` | `0x7f` | `BPF_PQR` | `BPF_UHMUL \| BPF_X` | v2 |
| `shmul64_imm` | `dst = ((i128)dst * (i128)imm)>>64` | `0x87` | `BPF_PQR` | `BPF_SHMUL \| BPF_K` | v2 |
| `shmul64_reg` | `dst = ((i128)dst * (i128)src)>>64` | `0x8f` | `BPF_PQR` | `BPF_SHMUL \| BPF_X` | v2 |
| `hor64_imm` | `dst = dst \| (imm << 32)` | `0x67`¹ | `BPF_PQR` | `BPF_HOR \| BPF_K` | v2 |

## ALU Instructions (32-bit)

| Mnemonic | Description | Opcode Hex | Class | Mode/Source | Version |
|----------|-------------|------------|-------|-------------|---------|
| `add32_imm` | `dst = (i32)dst + (i32)imm` | `0x04` | `BPF_ALU32_LOAD` | `BPF_ADD \| BPF_K` | All |
| `add32_reg` | `dst = (i32)dst + (i32)src` | `0x0c` | `BPF_ALU32_LOAD` | `BPF_ADD \| BPF_X` | All |
| `sub32_imm` | `dst = (i32)dst - (i32)imm` | `0x14` | `BPF_ALU32_LOAD` | `BPF_SUB \| BPF_K` | All |
| `sub32_reg` | `dst = (i32)dst - (i32)src` | `0x1c` | `BPF_ALU32_LOAD` | `BPF_SUB \| BPF_X` | All |
| `mul32_imm` | `dst = (i32)dst * (i32)imm` | `0x24` | `BPF_ALU32_LOAD` | `BPF_MUL \| BPF_K` | All |
| `mul32_reg` | `dst = (i32)dst * (i32)src` | `0x2c` | `BPF_ALU32_LOAD` | `BPF_MUL \| BPF_X` | All |
| `div32_imm` | `dst = (u32)dst / (u32)imm` | `0x34` | `BPF_ALU32_LOAD` | `BPF_DIV \| BPF_K` | All |
| `div32_reg` | `dst = (u32)dst / (u32)src` | `0x3c` | `BPF_ALU32_LOAD` | `BPF_DIV \| BPF_X` | All |
| `or32_imm` | `dst = (u32)dst \| (u32)imm` | `0x44` | `BPF_ALU32_LOAD` | `BPF_OR \| BPF_K` | All |
| `or32_reg` | `dst = (u32)dst \| (u32)src` | `0x4c` | `BPF_ALU32_LOAD` | `BPF_OR \| BPF_X` | All |
| `and32_imm` | `dst = (u32)dst & (u32)imm` | `0x54` | `BPF_ALU32_LOAD` | `BPF_AND \| BPF_K` | All |
| `and32_reg` | `dst = (u32)dst & (u32)src` | `0x5c` | `BPF_ALU32_LOAD` | `BPF_AND \| BPF_X` | All |
| `lsh32_imm` | `dst = (u32)dst << (imm & 31)` | `0x64` | `BPF_ALU32_LOAD` | `BPF_LSH \| BPF_K` | All |
| `lsh32_reg` | `dst = (u32)dst << (src & 31)` | `0x6c` | `BPF_ALU32_LOAD` | `BPF_LSH \| BPF_X` | All |
| `rsh32_imm` | `dst = (u32)dst >> (imm & 31)` | `0x74` | `BPF_ALU32_LOAD` | `BPF_RSH \| BPF_K` | All |
| `rsh32_reg` | `dst = (u32)dst >> (src & 31)` | `0x7c` | `BPF_ALU32_LOAD` | `BPF_RSH \| BPF_X` | All |
| `neg32` | `dst = -(i32)dst` | `0x84` | `BPF_ALU32_LOAD` | `BPF_NEG` | v1 |
| `mod32_imm` | `dst = (u32)dst % (u32)imm` | `0x94` | `BPF_ALU32_LOAD` | `BPF_MOD \| BPF_K` | All |
| `mod32_reg` | `dst = (u32)dst % (u32)src` | `0x9c` | `BPF_ALU32_LOAD` | `BPF_MOD \| BPF_X` | All |
| `xor32_imm` | `dst = (u32)dst ^ (u32)imm` | `0xa4` | `BPF_ALU32_LOAD` | `BPF_XOR \| BPF_K` | All |
| `xor32_reg` | `dst = (u32)dst ^ (u32)src` | `0xac` | `BPF_ALU32_LOAD` | `BPF_XOR \| BPF_X` | All |
| `mov32_imm` | `dst = (u32)imm` | `0xb4` | `BPF_ALU32_LOAD` | `BPF_MOV \| BPF_K` | All |
| `mov32_reg` | `dst = (u32)src` | `0xbc` | `BPF_ALU32_LOAD` | `BPF_MOV \| BPF_X` | All |
| `arsh32_imm` | `dst = (i32)dst >> (imm & 31)` | `0xc4` | `BPF_ALU32_LOAD` | `BPF_ARSH \| BPF_K` | All |
| `arsh32_reg` | `dst = (i32)dst >> (src & 31)` | `0xcc` | `BPF_ALU32_LOAD` | `BPF_ARSH \| BPF_X` | All |
| `sdiv32_imm` | `dst = (i32)dst / (i32)imm` | `0x14`¹ | `BPF_PQR` | `BPF_SDIV \| BPF_K` | v2 |
| `sdiv32_reg` | `dst = (i32)dst / (i32)src` | `0x1c`¹ | `BPF_PQR` | `BPF_SDIV \| BPF_X` | v2 |
| `srem32_imm` | `dst = (i32)dst % (i32)imm` | `0x24`¹ | `BPF_PQR` | `BPF_SREM \| BPF_K` | v2 |
| `srem32_reg` | `dst = (i32)dst % (i32)src` | `0x2c`¹ | `BPF_PQR` | `BPF_SREM \| BPF_X` | v2 |

## Memory Operations

| Mnemonic | Description | Opcode Hex | Class | Mode/Source | Version |
|----------|-------------|------------|-------|-------------|---------|
| `ldxb` | `dst = *(u8*)(src + off)` | `0x71` | `BPF_LDX` | `BPF_MEM \| BPF_B` | All |
| `ldxh` | `dst = *(u16*)(src + off)` | `0x69` | `BPF_LDX` | `BPF_MEM \| BPF_H` | All |
| `ldxw` | `dst = *(u32*)(src + off)` | `0x61` | `BPF_LDX` | `BPF_MEM \| BPF_W` | All |
| `ldxdw` | `dst = *(u64*)(src + off)` | `0x79` | `BPF_LDX` | `BPF_MEM \| BPF_DW` | All |
| `stb_imm` | `*(u8*)(dst + off) = imm` | `0x72` | `BPF_ST` | `BPF_MEM \| BPF_B` | All |
| `stb_reg` | `*(u8*)(dst + off) = src` | `0x73` | `BPF_STX` | `BPF_MEM \| BPF_B` | All |
| `sth_imm` | `*(u16*)(dst + off) = imm` | `0x6a` | `BPF_ST` | `BPF_MEM \| BPF_H` | All |
| `sth_reg` | `*(u16*)(dst + off) = src` | `0x6b` | `BPF_STX` | `BPF_MEM \| BPF_H` | All |
| `stw_imm` | `*(u32*)(dst + off) = imm` | `0x62` | `BPF_ST` | `BPF_MEM \| BPF_W` | All |
| `stw_reg` | `*(u32*)(dst + off) = src` | `0x63` | `BPF_STX` | `BPF_MEM \| BPF_W` | All |
| `stdw_imm` | `*(u64*)(dst + off) = imm` | `0x7a` | `BPF_ST` | `BPF_MEM \| BPF_DW` | All |
| `stdw_reg` | `*(u64*)(dst + off) = src` | `0x7b` | `BPF_STX` | `BPF_MEM \| BPF_DW` | All |
| `lddw` | `dst = imm` (64-bit) | `0x18` | `BPF_LD` | `BPF_DW \| BPF_IMM` | v2 |

## Jump Instructions

| Mnemonic | Description | Opcode Hex | Class | Mode/Source | Version |
|----------|-------------|------------|-------|-------------|---------|
| `ja` | Jump always | `0x05` | `BPF_JMP` | `BPF_JA` | All |
| `jeq_imm` | `PC += off if dst == imm` | `0x15` | `BPF_JMP` | `BPF_JEQ \| BPF_K` | All |
| `jeq_reg` | `PC += off if dst == src` | `0x1d` | `BPF_JMP` | `BPF_JEQ \| BPF_X` | All |
| `jgt_imm` | `PC += off if dst > imm (unsigned)` | `0x25` | `BPF_JMP` | `BPF_JGT \| BPF_K` | All |
| `jgt_reg` | `PC += off if dst > src (unsigned)` | `0x2d` | `BPF_JMP` | `BPF_JGT \| BPF_X` | All |
| `jge_imm` | `PC += off if dst >= imm (unsigned)` | `0x35` | `BPF_JMP` | `BPF_JGE \| BPF_K` | All |
| `jge_reg` | `PC += off if dst >= src (unsigned)` | `0x3d` | `BPF_JMP` | `BPF_JGE \| BPF_X` | All |
| `jlt_imm` | `PC += off if dst < imm (unsigned)` | `0xa5` | `BPF_JMP` | `BPF_JLT \| BPF_K` | All |
| `jlt_reg` | `PC += off if dst < src (unsigned)` | `0xad` | `BPF_JMP` | `BPF_JLT \| BPF_X` | All |
| `jle_imm` | `PC += off if dst <= imm (unsigned)` | `0xb5` | `BPF_JMP` | `BPF_JLE \| BPF_K` | All |
| `jle_reg` | `PC += off if dst <= src (unsigned)` | `0xbd` | `BPF_JMP` | `BPF_JLE \| BPF_X` | All |
| `jset_imm` | `PC += off if dst & imm` | `0x45` | `BPF_JMP` | `BPF_JSET \| BPF_K` | All |
| `jset_reg` | `PC += off if dst & src` | `0x4d` | `BPF_JMP` | `BPF_JSET \| BPF_X` | All |
| `jne_imm` | `PC += off if dst != imm` | `0x55` | `BPF_JMP` | `BPF_JNE \| BPF_K` | All |
| `jne_reg` | `PC += off if dst != src` | `0x5d` | `BPF_JMP` | `BPF_JNE \| BPF_X` | All |
| `jsgt_imm` | `PC += off if dst > imm (signed)` | `0x65` | `BPF_JMP` | `BPF_JSGT \| BPF_K` | All |
| `jsgt_reg` | `PC += off if dst > src (signed)` | `0x6d` | `BPF_JMP` | `BPF_JSGT \| BPF_X` | All |
| `jsge_imm` | `PC += off if dst >= imm (signed)` | `0x75` | `BPF_JMP` | `BPF_JSGE \| BPF_K` | All |
| `jsge_reg` | `PC += off if dst >= src (signed)` | `0x7d` | `BPF_JMP` | `BPF_JSGE \| BPF_X` | All |
| `jslt_imm` | `PC += off if dst < imm (signed)` | `0xc5` | `BPF_JMP` | `BPF_JSLT \| BPF_K` | All |
| `jslt_reg` | `PC += off if dst < src (signed)` | `0xcd` | `BPF_JMP` | `BPF_JSLT \| BPF_X` | All |
| `jsle_imm` | `PC += off if dst <= imm (signed)` | `0xd5` | `BPF_JMP` | `BPF_JSLE \| BPF_K` | All |
| `jsle_reg` | `PC += off if dst <= src (signed)` | `0xdd` | `BPF_JMP` | `BPF_JSLE \| BPF_X` | All |

## Byte Swap Instructions

| Mnemonic | Description | Opcode Hex | Class | Mode/Source | Size (imm) | Version |
|----------|-------------|------------|-------|-------------|------------|---------|
| `be16` | Convert to big endian (16-bit) | `0xd4` | `BPF_ALU` | `BPF_END \| BPF_TO_BE` | 16 | All |
| `be32` | Convert to big endian (32-bit) | `0xd4` | `BPF_ALU` | `BPF_END \| BPF_TO_BE` | 32 | All |
| `be64` | Convert to big endian (64-bit) | `0xd4` | `BPF_ALU` | `BPF_END \| BPF_TO_BE` | 64 | All |
| `le16` | Convert to little endian (16-bit) | `0xdc` | `BPF_ALU` | `BPF_END \| BPF_TO_LE` | 16 | v1 |
| `le32` | Convert to little endian (32-bit) | `0xdc` | `BPF_ALU` | `BPF_END \| BPF_TO_LE` | 32 | v1 |
| `le64` | Convert to little endian (64-bit) | `0xdc` | `BPF_ALU` | `BPF_END \| BPF_TO_LE` | 64 | v1 |

## Control Flow Instructions

| Mnemonic | Description | Opcode Hex | Class | Mode/Source | Version |
|----------|-------------|------------|-------|-------------|---------|
| `call_imm` | Function call (syscall) | `0x85` | `BPF_JMP` | `BPF_CALL` | All |
| `call_reg` | Function call via register | `0x8d` | `BPF_JMP` | `BPF_CALL \| BPF_X` | All |
| `exit` | Return from program | `0x95` | `BPF_JMP` | `BPF_EXIT` | All |
| `add_stk` | `sp += imm` (VM stack pointer). `dst` field in instruction must be 0. | `0x07`¹ | `BPF_ALU64_STORE` | `BPF_ADD \| BPF_K` | v2 |

## Instruction Encoding

All sBPF instructions use a fixed-size 64-bit encoding format (with the exception of `lddw` which uses 128 bits):

```
 MSB                                                        LSB
+--------+--------+--------+--------+--------+--------+--------+--------+
|immediate (32)                     |offset (16)    |src (4)|dst (4)|op (8)|
+--------+--------+--------+--------+--------+--------+--------+--------+
```

* `op` (8 bits): Operation code, determining the instruction type
* `dst` (4 bits): Destination register (R0-R10)
* `src` (4 bits): Source register (R0-R10)
* `offset` (16 bits): Memory offset or relative jump target
* `immediate` (32 bits): Immediate value operand

The `lddw` instruction spans two 64-bit slots to accommodate a full 64-bit immediate value.

---

¹ **Note on Opcode Overloading**: Several SBPFv2 instructions, particularly those categorized under "Class `BPF_PQR`" (e.g., `sdiv64_imm`, `srem32_reg`) and others like `hor64_imm`, `shmul64_imm`, and `add_stk`, reuse the same 8-bit hexadecimal opcode values as pre-existing SBPFv1 ALU operations. The Solana VM distinguishes these instructions based on the active SBPF version and specific feature flags (e.g., `enable_pqr_ops()`, `enable_lddw()`, `enable_add_stk_imm()`). For example, opcode `0x17` is interpreted as `sub64_imm` in SBPFv1, but as `sdiv64_imm` in SBPFv2 if PQR operations are enabled. Similarly, `add_stk` uses opcode `0x07` and is active in SBPFv2 if its feature flag is enabled and the instruction's `dst` register field is 0. Consult the specific SBPF version's verifier and interpreter logic for the precise disambiguation rules.

| [← Up: Appendices](./README.md) | [Next: Memory Map Reference →](./02_memory_map.md) |
|:-------------------------------:|----------------------------------------------------:|