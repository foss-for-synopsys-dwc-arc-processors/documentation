# Compatibility with MetaWare, nSIM and ARChitect

## Choosing a Processor Family

| Processor Family | CCAC/MDB | nSIM                          | GCC binary      | GCC option                                                                                           |
|------------------|----------|-------------------------------|-----------------|------------------------------------------------------------------------------------------------------|
| ARC HS6x         | `-arc64` | `nsim_isa_family=arc64`       | `arc64-elf-gcc` | [`-mcpu=` values for ARC HS6x](./target-options.md#arcv3-toolchain)                                  |
| ARC HS5x         | `-av3hs` | `nsim_isa_family=av3hs`       | `arc64-elf-gcc` | [`-mcpu=` values for ARC HS5x](./target-options.md#arcv3-toolchain)                                  |
| ARC HS3x/4x      | `-av2hs` | `nsim_isa_family=av2hs`       | `arc-elf32-gcc` | [`-mcpu=` values for ARC HS3x/4x](./target-options.md#values-of-mcpu-for-arc-hs3x-and-hs4x-families) |
| ARC EM           | `-av2em` | `nsim_isa_family=av2em`       | `arc-elf32-gcc` | [`-mcpu=` values for ARC EM](./target-options.md#values-of--mcpu-for-arc-em-family)                  |
| ARC700           | `-a7`    | `nsim_isa_family=a700`        | `arc-elf32-gcc` | [`-mcpu=` values for ARC700](./target-options.md#values-of--mcpu-for-arc-600-and-700-families)       |
| ARC600           | `-a6`    | `nsim_isa_family={a600,a601}` | `arc-elf32-gcc` | [`-mcpu=` values for ARC600](./target-options.md#values-of--mcpu-for-arc-600-and-700-families)       |

## Configuration Options

| Description                  | ARChitect                          | CCAC/MDB      | nSIM                             | GCC                  |
|------------------------------|------------------------------------|---------------|----------------------------------|----------------------|
| Shift options                | `-shift_option=3 -bitfield_option` | `-Xsa -Xbs`   | `nsim_isa_shift_option=3`        | `-mbarrel-shifter`   |
| Bitscan options              | `-bitscan_option`                  | `-Xnorm`      | `nsim_isa_bitscan_option=1`      | `-mnorm`             |
| Swap options                 | `-swap_option`                     | `-Xswap`      | `nsim_isa_swap_option=1`         | `-mswap`             |
| 16 register file             | `-rgf_num_regs=16`                 | N/A           | `nsim_isa_rgf_num_regs=16`       | `-rf16`              |
| Code density options         | `-code_density_option`             | `-Xcd`        | `nsim_isa_code_density_option=2` | `-mcode-density`     |
| DIV/REM options              | `-div_rem_option={1,2}`            | `-Xdiv_rem`   | `nsim_isa_div_rem_option={1,2}`  | `-mdiv-rem`          |
| Atomic options               | `-atomic_option`                   | `-Xatomic`    | `nsim_isa_atomic_option=1`       | `-matomic`           |
| 64-bit load and store        | `-ll64_option`                     | `-Xll64`      | `nsim_isa_ll64_option=64`        | `-mll64`             |
| Unaligned access             | N/A                                | `-Xunaligned` | `nsim_isa_unaligned_option=1`    | `-munaligned-access` |
| ARC 600 32x16 multiplication |                                    | `-Xmul32x16`  | `nsim_isa_mul32x16=1`            | `-mmul32x16`         |
| ARC 600 32x32 multiplication |                                    |               | `nsim_isa_mult32d=1`             | `-mmul64`            |
