# Memory Maps and Linker Scripts

## Linker Scripts and Linker Emulation

Linker script is a special file which specifies where to put different sections of ELF file
and defines particular symbols which may be referenced by an application. Linker emulation is
basically way to select one of the predetermined linker scripts of the GNU linker.
A linker script for a default linker emulation for ARCv2 may be observed this way:

```text
$ arc-elf32-ld --verbose
GNU ld (ARCompact/ARCv2 ISA elf32 toolchain - build 1360) 2.40.50.20230314
  Supported emulations:
   arcelf
   arclinux
   arclinux_nps
   arcv2elf
   arcv2elfx
using internal linker script:
==================================================
/* Default linker script, for normal executables */
OUTPUT_FORMAT("elf32-littlearc", "elf32-bigarc",
              "elf32-littlearc")
OUTPUT_ARCH(arc)
ENTRY(__start)

...

SECTIONS
{
  /* Read-only sections, merged into text segment: */
  PROVIDE (__executable_start = SEGMENT_START("text-segment", 0x100)); . = SEGMENT_START("text-segment", 0x100);
  .interp         : { *(.interp) }
  .hash           : { *(.hash) }
  .dynsym         : { *(.dynsym) }
  .dynstr         : { *(.dynstr) }
  .gnu.version    : { *(.gnu.version) }
  .gnu.version_d  : { *(.gnu.version_d) }
  .gnu.version_r  : { *(.gnu.version_r) }
  .rel.init       : { *(.rel.init) }
  .rela.init      : { *(.rela.init) }

...
```

Linux user-space applications are loaded by the dynamic linker in their own virtual memory address space,
where they do not collide with other applications. On the other side, baremetal applications are loaded into
target's memory by debugger, bootloader or they are already in the ROM mapped to a specific location.

If linker uses an invalid memory map for a particular platform, then some parts of the application will be loaded
to the memory incorrectly. For example, it may be accidentally written to peripherals' region and cause an error.

That default linker emulation places all loadable ELF sections in a row after each other starting at address `0x0`.
This is usually enough for an application prototyping, however real systems often have much more complex memory maps
with CCM regions, peripherals' region, etc.

Default linker emulation also puts interrupt vector table (`.ivt` section) between code and data sections and doesn't
align `.ivt` properly (`.ivt` must be 1KiB-aligned for ARC processors). Here is an example:

```text
$ arc-elf32-gcc -mcpu=em4_dmips main.c -o main.elf
$ arc-elf32-objdump -h main.elf

main.elf:     file format elf32-littlearc

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .init         00000022  00000100  00000100  00000100  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  1 .text         00003c28  00000124  00000124  00000124  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  2 .fini         00000016  00003d4c  00003d4c  00003d4c  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  3 .rodata       00000014  00003d64  00003d64  00003d64  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  4 .ivt          00000054  00003d78  00003d78  00003d78  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  5 .data         00000530  00005dcc  00005dcc  00003dcc  2**2
                  CONTENTS, ALLOC, LOAD, DATA
```

Therefore the default linker emulation is not applicable for applications which handle interrupts.
It can be used safely only with applications which don't handle interrupts and only on simulations
which simulate whole address space, like following templates: `em6_dmips`, `em6_gp`, `em6_mini`,
`em7d_nrg`, `em7d_voice_audio`, `em11d_nrg`, `em11d_voice_audio`, `hs36_base`, `hs36`, `hs38_base`,
`hs38`, `hs38_full`, `hs38_slc_full`.

## Using `arcv2elfx` linker emulation

If you use `arcv2elfx` linker emulation, then linker searches for `memory.x` file with definition of
a custom memory map. It is searched in the current working directory and in directories listed via
`-L` option.

### Example for EM Software Development Platform

Here is an example of `memory.x` for EM Software Development Platform:

```text
MEMORY
{
    ICCM0 : ORIGIN = 0x10000000, LENGTH = 128K
    PSRAM : ORIGIN = 0x40000000, LENGTH =  16M
    DCCM0 : ORIGIN = 0x80000000, LENGTH = 128K
}

REGION_ALIAS("startup", ICCM0)
REGION_ALIAS("text", ICCM0)
REGION_ALIAS("data", DCCM0)
REGION_ALIAS("sdata", DCCM0)

PROVIDE (__stack_top = (0x8001ffff & -4));
PROVIDE (__end_heap =  (0x8001ffff));
```

`MEMORY` section specifies platform's memory regions: base addresses and lengths.
You can use arbitrary names for these regions.
`REGION_ALIAS` commands translate platform's regions to standard region names
expected by the linker emulation. There are 4 such standard regions:

* `startup` - interrupt vector table and initialization code. By default it's mapped to `0x0`
  address and if you map `startup` to a different one, then you also need to pass this address
  to the linker using `--defsym=ivtbase_addr=<...>` option or to GCC itself using `-Wl,--defsym=ivtbase_addr=<...>` option.
* `text` - other code sections.
* `data` - data sections.
* `sdata` - small data sections.

Also, the example provides these symbols (both of them may be omitted and default values will be used):

* `__stack_top` points to the top of the stack. It must be 4-byte aligned In the example it points to the end
  of DRAM regions, because stack grows downward.
* `__end_heap` points to the end of the heap. Heap starts at the end of data sections
  and grows upward to `__end_heap`.

`startup` is mapped to `0x10000000`. It means that you have to pass `-Wl,--defsym=ivtbase_addr=<...>` option too.
You can compile your application against that `memory.x` file by passing `-marcv2elfx` to the linker or
`-Wl,-marcv2elfx` to GCC itself:

```text
$ ls
main.c memory.x
$ arc-elf32-gcc -mcpu=em4_fpuda -mmpy-option=6 -mfpu=fpuda_all \
                -Wl,-marcv2elfx -Wl,--defsym=ivtbase_addr=0x10000000 \
                main.c -o main.elf
```

### Example for HS Development Kit

Here is an example of `memory.x` for HS Development Kit:

```text
MEMORY
{
    DRAM : ORIGIN = 0x90000000, LENGTH = 0x50000000
}

REGION_ALIAS("startup", DRAM)
REGION_ALIAS("text", DRAM)
REGION_ALIAS("data", DRAM)
REGION_ALIAS("sdata", DRAM)
```

`startup` is mapped to `0x90000000`. It means that you have to pass `-Wl,--defsym=ivtbase_addr=<...>` option too.
You can compile your application against that `memory.x` this way:

```text
$ ls
main.c memory.x
$ arc-elf32-gcc -mcpu=archs -Wl,-marcv2elfx -Wl,--defsym=ivtbase_addr=0x90000000 \
                main.c -o main.elf
```

## Memory Maps for Development Platforms

### EM Software Development Platform

`memory.x` file for EM Software Development Platform v1.2:

```text
MEMORY
{
    ICCM0 : ORIGIN = 0x10000000, LENGTH = 128K
    PSRAM : ORIGIN = 0x40000000, LENGTH =  16M
    DCCM  : ORIGIN = 0x80000000, LENGTH = 128K
}

REGION_ALIAS("startup", ICCM0)
REGION_ALIAS("text", ICCM0)
REGION_ALIAS("data", DCCM)
REGION_ALIAS("sdata", DCCM)

PROVIDE (__stack_top = (0x8001ffff & -4));
PROVIDE (__end_heap =  (0x8001ffff));
```

`memory.x` file for EM Software Development Platform v1.0 and v1.1:

```text
MEMORY
{
    PSRAM : ORIGIN = 0x10000000, LENGTH =  16M
    ICCM0 : ORIGIN = 0x60000000, LENGTH = 128K
    DCCM  : ORIGIN = 0x80000000, LENGTH = 128K
}

REGION_ALIAS("startup", ICCM0)
REGION_ALIAS("text", ICCM0)
REGION_ALIAS("data", DCCM)
REGION_ALIAS("sdata", DCCM)

PROVIDE (__stack_top = (0x8001ffff & -4));
PROVIDE (__end_heap =  (0x8001ffff));
```

### HS Development Kit

Memory map for HS Development Kit and HS Development Kit 4xD depends on a board
itself and on a chosen configuration of CPUs. Consider using this `memory.x` file
that fits for all configurations (refer to [HSDK user guide](../../platforms/board-hsdk.md)
or [HSDK 4xD user guide](../../platforms/board-hsdk-4xd.md) for details about board's memory map):

```text
MEMORY
{
    DRAM : ORIGIN = 0x90000000, LENGTH = 0x50000000
}

REGION_ALIAS("startup", DRAM)
REGION_ALIAS("text", DRAM)
REGION_ALIAS("data", DRAM)
REGION_ALIAS("sdata", DRAM)
```

### IoT Development Kit

`memory.x` file for IoT Development Kit:

```text
MEMORY
{
    ICCM : ORIGIN = 0x20000000, LENGTH = 256K
    DCCM : ORIGIN = 0x80000000, LENGTH = 128K
}

REGION_ALIAS("startup", ICCM)
REGION_ALIAS("text", ICCM)
REGION_ALIAS("data", DCCM)
REGION_ALIAS("sdata", DCCM)

PROVIDE (__stack_top = (0x8001ffff & -4));
PROVIDE (__end_heap =  (0x8001ffff));
```

### EM Starter Kit

`memory.x` file for EM Starter Kit v2.2 and v2.3 with EM11D core:

```text
MEMORY
{
    ICCM : ORIGIN = 0x00000000, LENGTH =  64K
    DRAM : ORIGIN = 0x10000000, LENGTH = 128M
    DCCM : ORIGIN = 0x80000000, LENGTH =  64K
}

REGION_ALIAS("startup", ICCM)
REGION_ALIAS("text", ICCM)
REGION_ALIAS("data", DRAM)
REGION_ALIAS("sdata", DRAM)

PROVIDE (__stack_top = (0x17FFFFFF & -4));
PROVIDE (__end_heap = (0x17FFFFFF));
```

`memory.x` file for EM Starter Kit v2.2 and v2.3 with EM7D and EM9D cores:

```text
MEMORY
{
    ICCM : ORIGIN = 0x00000000, LENGTH = 256K
    DRAM : ORIGIN = 0x10000000, LENGTH = 128M
    DCCM : ORIGIN = 0x80000000, LENGTH = 128K
}

REGION_ALIAS("startup", ICCM)
REGION_ALIAS("text", ICCM)
REGION_ALIAS("data", DRAM)
REGION_ALIAS("sdata", DRAM)

PROVIDE (__stack_top = (0x17FFFFFF & -4));
PROVIDE (__end_heap = (0x17FFFFFF));
```

`memory.x` file for EM Starter Kit v2.1 with EM7D core:

```text
MEMORY
{
    ICCM : ORIGIN = 0x00000000, LENGTH =  32K
    DRAM : ORIGIN = 0x10000000, LENGTH = 128M
    DCCM : ORIGIN = 0x80000000, LENGTH =  32K
}

REGION_ALIAS("startup", ICCM)
REGION_ALIAS("text", ICCM)
REGION_ALIAS("data", DRAM)
REGION_ALIAS("sdata", DRAM)

PROVIDE (__stack_top = (0x17FFFFFF & -4));
PROVIDE (__end_heap = (0x17FFFFFF));
```

`memory.x` file for EM Starter Kit v2.1 with EM5D core:

```text
MEMORY
{
    ICCM : ORIGIN = 0x00000000, LENGTH = 128K
    DCCM : ORIGIN = 0x80000000, LENGTH = 256K
}

REGION_ALIAS("startup", ICCM)
REGION_ALIAS("text", ICCM)
REGION_ALIAS("data", DCCM)
REGION_ALIAS("sdata", DCCM)

PROVIDE (__stack_top = (0x8003FFFF & -4));
PROVIDE (__end_heap = (0x8003FFFF));
```

`memory.x` file for EM Starter Kit v1 with EM6GP core:

```text
MEMORY
{
    ICCM : ORIGIN = 0x00000000, LENGTH =  32K
    DRAM : ORIGIN = 0x10000000, LENGTH = 128M
}

REGION_ALIAS("startup", ICCM)
REGION_ALIAS("text", ICCM)
REGION_ALIAS("data", DRAM)
REGION_ALIAS("sdata", DRAM)

PROVIDE (__stack_top = (0x17FFFFFF & -4));
PROVIDE (__end_heap = (0x17FFFFFF));
```

`memory.x` file for EM Starter Kit v1 with EM4 core:

```text
MEMORY
{
    ICCM : ORIGIN = 0x00000000, LENGTH = 128K
    DCCM : ORIGIN = 0x80000000, LENGTH =  64K
}

REGION_ALIAS("startup", ICCM)
REGION_ALIAS("text", ICCM)
REGION_ALIAS("data", DCCM)
REGION_ALIAS("sdata", DCCM)

PROVIDE (__stack_top = (0x8000FFFF & -4));
PROVIDE (__end_heap = (0x8000FFFF));
```

## Useful Links

You can find valid memory mappings for particular hardware platforms in documentation.
Here is a list of resources with memory maps for Synopsys' development platforms:

* [Development platforms](../../platforms/index.md) section contains documentation for
  all Synopsys' development platforms. User guides contain descriptions of memory mappings.
* [Newlib](https://github.com/foss-for-synopsys-dwc-arc-processors/newlib/tree/arc64) repository for ARC contains predefined
  memory maps for some of development platforms  in `libgloss/arc` directory.
* [toolchain](https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain) repository also contains predefined
  memory maps for some of development platforms in `extras/dev_systems` directory.
