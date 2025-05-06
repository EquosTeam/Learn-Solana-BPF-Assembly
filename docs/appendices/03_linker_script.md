# Sample Linker Script

This appendix provides a sample linker script template for sBPF programs. This script defines how program sections are arranged in memory during the build process.

## Standard Linker Script Template

A minimal script often looks like this, placing code and read-only data together:

```ld
PHDRS
{
  text PT_LOAD FLAGS(5);   /* R+X segment for Code + Read-Only Data */
}

SECTIONS
{
  /* Place sections relative to the start defined by --image-base */
  . = SIZEOF_HEADERS;
  .text : { *(.text*) } :text
  .rodata : { *(.rodata*) } :text /* Include rodata in the text segment */

  /* Discard sections not needed for typical static sBPF programs */
  /DISCARD/ : {
    *(.eh_frame*)
    *(.note*)
    *(.comment*)
    *(.dynamic*)
    *(.dynsym*)
    *(.dynstr*)
    *(.rela.dyn*)
    *(.reloc*)
    /* Add other sections to discard as needed */
  }
}

ENTRY(entrypoint) /* Define the entry point symbol */
```

## Script Breakdown

The linker script typically consists of:

### PHDRS (Program Headers)

Defines the program headers that describe memory segments. For simple sBPF, often only one `PT_LOAD` segment is needed for the code and read-only data, marked Read+Execute (`FLAGS(5)`, where 5 = Read (4) + Execute (1)).

### SECTIONS

Defines how input sections (`.text`, `.rodata`, etc. from object files) are mapped to output sections and segments:

-   `. = SIZEOF_HEADERS;`: Starts placing sections after the ELF headers.
-   `.text : { *(.text*) } :text`: Places all input `.text` sections into the output `.text` section, assigned to the `text` segment.
-   `.rodata : { *(.rodata*) } :text`: Places input `.rodata` sections also into the `text` segment.
-   `/DISCARD/`: Explicitly removes sections not usually required for sBPF deployment, reducing final binary size.
-   `ENTRY(symbol)`: Specifies the function symbol (e.g., `entrypoint`) where program execution begins.

## Usage

1.  Save this template as `program.ld` (or similar) in your project.
2.  Use it with the `ld.lld` linker command, specifying the image base:

```sh
# Compile assembly or C code
clang -target bpfel -march=sbf -c program.s -o program.o

# Link using the script and setting the base address
ld.lld -z notext -shared --Bstatic --entry entrypoint \
       --image-base 0x100000000 -T program.ld -o program.so program.o
```

*Key Linker Flags*:*
-   `--image-base 0x100000000`: Sets the starting virtual address for the loadable segment(s), crucial for sBPF.
-   `-T program.ld`: Specifies the linker script to use.
-   `-shared`: Creates a relocatable shared object file (.so), the format expected by the Solana loader.
-   `--entry entrypoint`: Redundant if `ENTRY()` is in the script, but good practice.
-   `-z notext`: Disables warnings about text segment being writable (not typically needed).
-   `--Bstatic`: Prefer static libraries if linking against any (not common for pure assembly).

## Customization

-   If you have initialized data (`.data`) or uninitialized data (`.bss`), you might need to define a separate `PT_LOAD` segment with Read+Write flags (`FLAGS(6)`, where 6 = Read (4) + Write (2)) and map these sections accordingly, though static data is less common in simple sBPF programs.
-   Alignment can be specified using `ALIGN()` within section definitions if needed.

---

| [← Previous: Memory Map Reference](./02_memory_map.md) | [Up: Appendices](./README.md) | [Next: Compiler and Linker Flags →](./04_build_flags.md) |
|:------------------------------------------------------:|:-----------------------------:|:---------------------------------------------------------:|
  