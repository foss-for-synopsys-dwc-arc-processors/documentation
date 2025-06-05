# Building applications with Newlib

!!! warning

    Newlib is considered as deprecated standard library for ARC-V
    GNU toolchain. It will be completely removed in the future
    releases. Please, consider migrating to
    [the Picolibc-based toolchain](./building-picolibc.md).

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

## Getting started

Consider a simple code example:

```c
int main() {
    return 0;
}
```

If I want to build it for the base RMX-100 target, I would use this set of options:

```
$ riscv64-snps-elf-gcc \
        -march=rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr \
        -mabi=ilp32 \
        -mtune=arc-v-rmx-100-series \
        example.c -o example.elf
```

By default, GCC links applications with `libgloss.a` library that provides a set
of `ecall` based input/output capabilities like `printf`, `fopen`, etc. To link an
application with `ebreak` based `libsemihost.a` library use `-specs=semihost.specs`:

```
$ riscv64-snps-elf-gcc \
        -march=rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr \
        -mabi=ilp32 \
        -mtune=arc-v-rmx-100-series \
        -specs=semihost.specs \
        example.c -o example.elf
```

Refer to [Running on nSIM](./nsim.md) and [Running on QEMU](./qemu.md) to learn how
to run ARC-V examples on nSIM or QEMU simulator.

## Using custom linker script

GNU toolchain for ARC-V is shipped with a custom ARC-V specific startup code
and a custom linker script. They are intended to be used in pair by passing
`-specs=arcv.specs -T arcv.ld` to GCC. Here is an example:

```
$ riscv64-snps-elf-gcc \
        -march=rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr \
        -mabi=ilp32 \
        -mtune=arc-v-rmx-100-series \
        -specs=semihost.specs \
        -specs=arcv.specs \
        -T arcv.ld \
        example.c -o example.elf
```

Hot does it differ from using the standard Newlib startup code and the standard
GCC linker script:

1. ARC-V caches are turned on startup.
2. Stack is initialized correctly (the standard Newlib startup code does not initialize it).
3. Command line arguments are processed through Semihosting interface if an application
   is linked with `-specs=semihost.specs` option.

When `arcv.ld` linker script is used, base address and size of code or data section
may be set through `-Wl,-defsym=` option:

1. `-Wl,-defsym=txtmem_addr=0x80000000` - set `0x80000000` as base address for code sections.
2. `-Wl,-defsym=txtmem_len=1M` - set code sections size to 1MB.
3. `-Wl,-defsym=datamem_addr=0x80200000` - set `0x80200000` as base address for data sections.
4. `-Wl,-defsym=datamem_len=1M` - set code sections size to 1MB.

Full command line with custom placement of code and data section:

```
$ riscv64-snps-elf-gcc \
        -march=rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr \
        -mabi=ilp32 \
        -mtune=arc-v-rmx-100-series \
        -Wl,-defsym=txtmem_addr=0x80000000 \
        -Wl,-defsym=txtmem_len=1M \
        -Wl,-defsym=datamem_addr=0x80200000 \
        -Wl,-defsym=datamem_len=1M \
        -specs=semihost.specs \
        -specs=arcv.specs \
        -T arcv.ld \
        example.c -o example.elf
```

Using custom code and data sections is essential for 64-bit targets. By default,
`arcv.ld` linker script puts code to `0x0` and data to `0x80000000`. However,
this memory layout may be not acceptable for 64-bit targets. That is why in
the GNU toolchain 64-bit targets are available only with `medany` memory model.

## Using startup code without CSRs

You can pass `--crt0=no-csr` option to choose a startup code without
CSRs when `-specs=arcv.specs` is passed:

```
$ riscv64-snps-elf-gcc \
        -march=rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr \
        -mabi=ilp32 \
        -mtune=arc-v-rmx-100-series \
        -specs=semihost.specs \
        -specs=arcv.specs \
        --crt0=no-csr \
        -T arcv.ld \
        example.c -o example.elf
```

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

Note that all `-mtune` values for ARC-V assume that fast unaligned access is supported
(`__riscv_misaligned_fast == 1`) by targets. This assumption may be overwritten by
`-mstrict-align` when building an application or toolchain libraries itself.
