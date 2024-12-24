# Building Applications

GNU toolchain for ARC-V targets uses `riscv64-snps-elf` prefix for
all tools. For example, GCC binary has name `riscv64-snps-elf-gcc`.

Usually, to compile an application you need to set target options:

1. `-march` - stands for RISC-V ISA.
2. `-mabi` - stands for ABI.
3. `-mtune` - stands for a particular GCC instruction scheduling optimization, 
   refer [Tuning Instruction Scheduling](#tuning-instruction-scheduling)
   for ARC-V specific values .
4. `-mcmodel` - `medlow` for `rv32` targets and `medany` for `rv64` targets.

All together they  correspond to a particular prebuilt standard library. Refer
[Understanding ARC-V Configurations](#understanding-arc-v-configurations) for details.

## Getting Started

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

## Using Custom Linker Script

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

## Understanding ARC-V Configurations

GNU toolchain for ARC-V is shipped with a set of prebuilt libraries for each
available configuration. A configuration is a combination of `-march`, `-mabi`,
`-mtune` and `-mcmodel` values. The default value for `-mtune` (if it's not passed
explicitly) is `arc-v-rmx-100-series` and the default value for `-mcmodel` is
`medlow`.

Here is a list of all combinations of options for which prebuilt libraries are
presented in the toolchain:

| ARC-V profile                               | `-march`                                               | `-mabi`  | `-mtune`                   | `-mcmodel`   |
|---------------------------------------------|--------------------------------------------------------|----------|----------------------------|--------------|
| Base RMX-100 mini                           | `rv32e`                                                | `ilp32e` | `arc-v-rmx-100-series`     | `medlow`     |
| Base RMX-100 mini + large set of extensions | `rv32emac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zdinx_zicsr` | `ilp32e` | `arc-v-rmx-100-series`     | `medlow`     |
| Base RMX-100                                | `rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr`               | `ilp32`  | `arc-v-rmx-100-series`     | `medlow`     |
| Base RMX-100 + `m`                          | `rv32im_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr`               | `ilp32`  | `arc-v-rmx-100-series`     | `medlow`     |
| Base RMX-100 + `mc`                         | `rv32imc_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr`              | `ilp32`  | `arc-v-rmx-100-series`     | `medlow`     |
| Base RMX-100 + `mac`                        | `rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr`             | `ilp32`  | `arc-v-rmx-100-series`     | `medlow`     |
| Base RMX-100 + `mac_zfinx`                  | `rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zicsr`       | `ilp32`  | `arc-v-rmx-100-series`     | `medlow`     |
| Base RMX-100 + `mac_zfinx_zdinx`            | `rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zdinx_zicsr` | `ilp32`  | `arc-v-rmx-100-series`     | `medlow`     |
| Base RMX-500                                | `rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr`               | `ilp32`  | `arc-v-rmx-500-series`     | `medlow`     |
| Base RMX-500 + `m`                          | `rv32im_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr`               | `ilp32`  | `arc-v-rmx-500-series`     | `medlow`     |
| Base RMX-500 + `mc`                         | `rv32imc_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr`              | `ilp32`  | `arc-v-rmx-500-series`     | `medlow`     |
| Base RMX-500 + `mac`                        | `rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr`             | `ilp32`  | `arc-v-rmx-500-series`     | `medlow`     |
| Base RMX-500 + `mac_zfinx`                  | `rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zicsr`       | `ilp32`  | `arc-v-rmx-500-series`     | `medlow`     |
| Base RMX-500 + `mac_zfinx_zdinx`            | `rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zdinx_zicsr` | `ilp32`  | `arc-v-rmx-500-series`     | `medlow`     |
| Base RHX-100                                | `rv32imac_zcb_zcmp_zba_zbb_zbs_zicsr`                  | `ilp32`  | `arc-v-rhx-100-series`     | `medlow`     |
| Base RHX-100 + `f`                          | `rv32imafc_zcb_zcmp_zba_zbb_zbs_zicsr`                 | `ilp32f` | `arc-v-rhx-100-series`     | `medlow`     |
| Base RHX-100 + `fd`                         | `rv32imafd_zca_zcb_zcmp_zba_zbb_zbs_zicsr`[^1]         | `ilp32d` | `arc-v-rhx-100-series`     | `medlow`     |
| Base RPX-100                                | `rv64imac_zcb_zba_zbb_zbs_zicsr`                       | `lp64`   | `arc-v-rmx-100-series`[^2] | `medany[^3]` |
| Base RPX-100 + `fd`                         | `rv64imafdc_zcb_zba_zbb_zbs_zicsr`                     | `lp64d`  | `arc-v-rmx-100-series`[^2] | `medany`     |

[^1]: Note that this configuration does not include `zcd` in `-march` since it's not supported by RHX-100.
[^2]: There is no `-mtune` option for RPX-100 targets in the GNU toolchain yet. Thus, a default `-mtune` value is used.
[^3]: In the GNU toolchain 64-bit targets are available only for `-mcmodel=medany` memory model.

Here is a list of fallback configurations that don't match any ARC-V profile:

| `-march`           | `-mabi`  | `-mtune`               | `-mcmodel` |
|--------------------|----------|------------------------|------------|
| `rv32i`            | `ilp32`  | `arc-v-rmx-100-series` | `medlow`   |
| `rv32i`            | `ilp32`  | `arc-v-rmx-500-series` | `medlow`   |
| `rv32i`            | `ilp32`  | `arc-v-rhx-100-series` | `medlow`   |
| `rv32ia`           | `ilp32`  | `arc-v-rmx-100-series` | `medlow`   |
| `rv32ia`           | `ilp32`  | `arc-v-rmx-500-series` | `medlow`   |
| `rv32ia`           | `ilp32`  | `arc-v-rhx-100-series` | `medlow`   |
| `rv32imc_zicsr`    | `ilp32`  | `arc-v-rmx-100-series` | `medlow`   |
| `rv32imc_zicsr`    | `ilp32`  | `arc-v-rmx-500-series` | `medlow`   |
| `rv32iac_zicsr`    | `ilp32`  | `arc-v-rmx-100-series` | `medlow`   |
| `rv32iac_zicsr`    | `ilp32`  | `arc-v-rmx-500-series` | `medlow`   |
| `rv32imac_zicsr`   | `ilp32`  | `arc-v-rmx-100-series` | `medlow`   |
| `rv32imac_zicsr`   | `ilp32`  | `arc-v-rmx-500-series` | `medlow`   |
| `rv32imac_zicsr`   | `ilp32`  | `arc-v-rhx-100-series` | `medlow`   |
| `rv32imafc_zicsr`  | `ilp32f` | `arc-v-rhx-100-series` | `medlow`   |
| `rv32imafdc_zicsr` | `ilp32d` | `arc-v-rhx-100-series` | `medlow`   |
| `rv64i`            | `lp64`   | `arc-v-rmx-100-series` | `medany`   |
| `rv64ia`           | `lp64`   | `arc-v-rmx-100-series` | `medany`   |
| `rv64imac_zicsr`   | `lp64`   | `arc-v-rmx-100-series` | `medany`   |
| `rv64imafdc`       | `lp64d`  | `arc-v-rmx-100-series` | `medany`   |

On linkage stage, when a set of target options is passed, GCC tries to select standard libraries
that match this set of options. If you are not sure what prebuilt libraries are selected for linking
with an application, you can use `-print-multi-os-directory` option to check this:

```
$ riscv64-snps-elf-gcc -march=rv32imac_zcb_zcmp_zba_zbb_zbs_zicsr -mabi=ilp32 -mtune=arc-v-rhx-100-series -print-multi-os-directory
rv32imac_zcb_zcmp_zba_zbb_zbs_zicsr/ilp32/rhx100
```

If `zcb` is removed from `-march`, then GCC selects a configuration that may be compatible with
this set of options:

```
$ riscv64-snps-elf-gcc -march=rv32imac_zcmp_zba_zbb_zbs_zicsr -mabi=ilp32 -mtune=arc-v-rhx-100-series -print-multi-os-directory
rv32imac_zicsr/ilp32/rhx100
```

## Tuning Instruction Scheduling

GCC instruction scheduling may be tuned for different ARC-V
targets using `-mtune=` option:

| Feature                  | Option                        |
|--------------------------|-------------------------------|
| Tune for RMX-100 targets | `-mtune=arc-v-rmx-100-series` |
| Tune for RMX-500 targets | `-mtune=arc-v-rmx-500-series` |
| Tune for RHX-100 targets | `-mtune=arc-v-rhx-100-series` |

For RMX-100 targets it's also possible to choose a version of MPY unit using `-param=arcv-mpy-option=` option:

* `-param=arcv-mpy-option=1c`
* `-param=arcv-mpy-option=2c` (default)
* `-param=arcv-mpy-option=10c`

Note that all `-mtune` values for ARC-V assume that fast unaligned access is supported
(`__riscv_misaligned_fast == 1`) by targets. This assumption may be overwritten by
`-mstrict-align` when building an application or toolchain libraries itself.
