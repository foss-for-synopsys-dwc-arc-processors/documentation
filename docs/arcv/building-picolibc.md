# Building applications with Picolibc

GNU toolchain for ARC-V targets uses `riscv64-snps-elf` prefix for
all tools. For example, GCC binary has name `riscv64-snps-elf-gcc`.

Usually, to compile an application you need to set target options:

1. `-march` - stands for RISC-V ISA.
2. `-mabi` - stands for ABI.
3. `-mtune` - stands for a particular GCC instruction scheduling optimization, 
   refer [Tuning Instruction Scheduling](#tuning-instruction-scheduling)
   for ARC-V specific values .
4. `-mcmodel` - `medlow` for `rv32` targets and `medany` for `rv64` targets.

All together they correspond to a particular prebuilt standard library. Refer
[Understanding ARC-V configurations](./multilib.md) for details.

## Getting Started

Consider a simple code example:

```c
#include <stdio.h>

int main()
{
        printf("Hello, World!\n");
        return 0;
}
```

If I want to build it for the base RMX-100 target and link with 
a semihosting library, I would use this set of options:

```
$ riscv64-snps-elf-gcc \
        -march=rv32imafc \
        -mabi=ilp32f \
        -mtune=arc-v-rhx-100-series \
        -specs=picolibc.specs \
        --crt0=semihost \
        --oslib=semihost \
        example.c -o example.elf
```

Passing `-specs=picolibc.specs` for linking with Picolibc is mandatory:

1. It allows choosing a proper variant of `crt0` startup file through `--crt0=` option ([see all variants below](#choosing-a-crt0-variant)).
2. It allows choosing a proper variant of a system library through `--oslib=` option ([see all variants below](#choosing-a-system-library)).
3. It chooses a custom linker script crafted for Picolibc (`picolibc.ld`).

In turn, when `picolibc.ld` linker script is used, it's possible to configure
placement and size of code and data sections:

1. Set offset of code sections through `-Wl,--defsym=__flash=` option.
2. Set size of code sections through `-Wl,--defsym=__flash_size=` option.
3. Set offset of data sections through `-Wl,--defsym=__ram=` option.
4. Set size of data sections through `-Wl,--defsym=__ram_size=` option.

Here is a full example with custom code and data sections:

```
$ riscv64-snps-elf-gcc \
        -march=rv32imafc \
        -mabi=ilp32f \
        -mtune=arc-v-rhx-100-series \
        -specs=picolibc.specs \
        --crt0=semihost \
        --oslib=semihost \
        -Wl,--defsym=__flash=0x80000000 \
        -Wl,--defsym=__flash_size=128K \
        -Wl,--defsym=__ram=0x80020000 \
        -Wl,--defsym=__ram_size=128K \
        example.c -o example.elf
```

Run the application using nSIM:

```
$ nsimdrv -p nsim_isa_family=rv32 -p nsim_isa_ext=-all.i.m.a.f.c.zicsr -p nsim_semihosting=1 -p enable_exceptions=0 example.elf
Hello, World!
```

Refer to [Running on nSIM](./nsim.md) and [Running on QEMU](./qemu.md) to learn how
to run ARC-V examples on nSIM or QEMU simulator.

## Choosing a `crt0` variant

A variant of a startup file (`crt0.o`) is chosen through `--crt0=` option. See a list of all supported values for this option below.

| Option                                | Uses CSR instructions | Returns exit code | Calls constructors | Reads command line arguments | Sets traps | Initializes ARC-V features |
|---------------------------------------|-----------------------|-------------------|--------------------|------------------------------|------------|----------------------------|
| Not passing `--crt0=` value (default) | No                    | No                | __Yes__            | No                           | No         | No                         |
| `--crt0=hosted`                       | No                    | __Yes__           | __Yes__            | No                           | No         | No                         |
| `--crt0=minimal`                      | No                    | No                | No                 | No                           | No         | No                         |
| `--crt0=semihost`                     | No                    | __Yes__           | __Yes__            | __Yes__                      | No         | No                         |
| `--crt0=semihost-trap`                | __Yes__               | __Yes__           | __Yes__            | __Yes__                      | __Yes__    | No                         |
| `--crt0=arcv-hosted`                  | __Yes__               | __Yes__           | __Yes__            | No                           | No         | __Yes__                    |
| `--crt0=arcv-minimal`                 | __Yes__               | No                | No                 | No                           | No         | __Yes__                    |
| `--crt0=arcv-semihost`                | __Yes__               | __Yes__           | __Yes__            | __Yes__                      | No         | __Yes__                    |
| `--crt0=arcv-semihost-trap`           | __Yes__               | __Yes__           | __Yes__            | __Yes__                      | __Yes__    | __Yes__                    |

Empty `crt0` variant corresponds to `--crt0=none`. All `arcv-*` variants enable all available ARC-V caches
and support of RVV if it's present.

## Choosing a system library

So far, Picolibc supports 2 system libraries, and it's chosen through `--oslib=` option:

| Option              | Description                                               |
|---------------------|-----------------------------------------------------------|
| `--oslib=dummyhost` | A simple environment without input/output and exit codes. |
| `--oslib=semihost`  | Support of Semihosting environment.                       |

## Tuning instruction scheduling

GCC instruction scheduling may be tuned for different ARC-V
targets using `-mtune=` option:

| Feature                  | Option                        |
|--------------------------|-------------------------------|
| Tune for RMX-100 targets | `-mtune=arc-v-rmx-100-series` |
| Tune for RMX-500 targets | `-mtune=arc-v-rmx-500-series` |
| Tune for RHX-100 targets | `-mtune=arc-v-rhx-100-series` |
| Tune for RPX-100 targets | `-mtune=arc-v-rpx-100-series` |

For RMX-100 targets it's also possible to choose a version of MPY unit using `-param=arcv-mpy-option=` option:

* `-param=arcv-mpy-option=1c`
* `-param=arcv-mpy-option=2c` (default)
* `-param=arcv-mpy-option=10c`

You can choose a number of cycles that a word-size integer load operation takes
(from 1 to 3, 3 is default) using `--param=arcv-ld-cycles=<1,3>` option.

Note that all `-mtune` values for ARC-V assume that fast unaligned access is supported
(`__riscv_misaligned_fast == 1`) by targets. This assumption may be overwritten by
`-mstrict-align` when building an application or toolchain libraries itself.


## Migrating from Newlib

### Build with semihosting and ARC-V features

Using Newlib:

```
...
-specs=semihost.specs \
-specs=arcv.specs \
-T arcv.ld \
...
```

Using Picolibc:

```
...
-specs=picolibc.specs \
--oslib=semihost \
--crt0=arcv-semihost \
...
```

### Build with semihosting and without ARC-V features

Using Newlib:

```
...
-specs=semihost.specs \
-specs=arcv.specs \
-T arcv.ld \
--crt0=no-csr \
...
```

Using Picolibc:

```
...
-specs=picolibc.specs \
--oslib=semihost \
--crt0=semihost \
...
```

### Set offset and size of text and data sections

Using Newlib:

```
...
-specs=semihost.specs \
-specs=arcv.specs \
-T arcv.ld \
-Wl,-defsym=txtmem_addr=0x80000000 \
-Wl,-defsym=txtmem_len=128K \
-Wl,-defsym=datamem_addr=0x80020000 \
-Wl,-defsym=datamem_len=128K \
...
```

Using Picolibc:

```
...
-specs=picolibc.specs \
--oslib=semihost \
--crt0=arcv-semihost \
-Wl,--defsym=__flash=0x80000000 \
-Wl,--defsym=__flash_size=128K \
-Wl,--defsym=__ram=0x80020000 \
-Wl,--defsym=__ram_size=128K \
...
```
