# sBPF Assembly Documentation – TODO Checklist

> Track progress while producing the first comprehensive Solana sBPF assembly guide.

## Legend
- [ ] **TODO** – not started
- [~] **WIP** – in progress
- [x] **Done** – complete

---

## 0. Project Setup
- [x] Decide documentation format (Markdown)
- [x] Bootstrap repo structure (docs/, examples/, build scripts)
- [x] Configure CI for spell-check, lint, and sample builds (Skipped)

## I – Foundations
- [x] Write "What is sBPF?" introduction
- [x] Describe differences between eBPF and Solana's sBPF
- [x] Draft tool-chain & environment section
  - [x] LLVM/Clang install instructions
  - [x] Linker script explanation
  - [x] `cargo build-sbf` vs manual clang
- [x] Document execution model
  - [x] Register set & calling convention
  - [x] Fixed memory map diagram
  - [x] Entry/exit semantics
- [x] Fully specify **Input Buffer Anatomy** *(requires research)*

## II – Assembly Language Reference
- [x] Draft GAS syntax primer
- [x] Compile opcode tables from `solana_sbpf::ebpf`
- [x] Document ALU, jump, load/store, byte-swap instructions
- [x] Provide encoding diagram (optional deep dive)

## III – Solana Integration & Practical Usage
- [ ] Produce **Helper / Syscall Catalogue**
  - [ ] Extract IDs & signatures from Solana runtime source
- [ ] Write Hello-World walkthrough
- [ ] Explain build & deploy (CLI / test-validator)
- [ ] Add cookbook examples (noop, fibonacci, etc.)
- [ ] Section on testing & debugging

## IV – Advanced Topics
- [ ] Interacting with accounts in assembly
- [ ] Cross-Program Invocation (CPI)
- [ ] Program Derived Addresses (PDAs)
- [ ] Heap usage via `sol_alloc_free`
- [ ] Performance & size optimizations
- [ ] Security considerations

## V – Appendices
- [x] Complete opcode reference table
- [x] Full helper ID table
- [x] Sample linker script template
- [x] Common clang & ld flag matrix

---

## Global Action Items
- [ ] 🔍 Gather authoritative helper list + IDs
- [x] 🔍 Reverse-engineer & document input buffer struct
- [ ] 🔍 Confirm canonical clang/ld flags for latest Solana release
- [ ] 🔍 Automate opcode table generation from crate constants
- [ ] 🔍 Create minimal CI sample that assembles & deploys `noop` 