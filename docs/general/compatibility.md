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

## General Configuration Options

| Description           | ARChitect                          | CCAC/MDB      | nSIM                             | GCC                  |
|-----------------------|------------------------------------|---------------|----------------------------------|----------------------|
| Shift options         | `-shift_option=3 -bitfield_option` | `-Xsa -Xbs`   | `nsim_isa_shift_option=3`        | `-mbarrel-shifter`   |
| Bitscan options       | `-bitscan_option`                  | `-Xnorm`      | `nsim_isa_bitscan_option=1`      | `-mnorm`             |
| Swap options          | `-swap_option`                     | `-Xswap`      | `nsim_isa_swap_option=1`         | `-mswap`             |
| 16 register file      | `-rgf_num_regs=16`                 | N/A           | `nsim_isa_rgf_num_regs=16`       | `-rf16`              |
| Code density options  | `-code_density_option`             | `-Xcd`        | `nsim_isa_code_density_option=2` | `-mcode-density`     |
| DIV/REM options       | `-div_rem_option={1,2}`            | `-Xdiv_rem`   | `nsim_isa_div_rem_option={1,2}`  | `-mdiv-rem`          |
| Atomic options        | `-atomic_option`                   | `-Xatomic`    | `nsim_isa_atomic_option=1`       | `-matomic`           |
| 64-bit load and store | `-ll64_option`                     | `-Xll64`      | `nsim_isa_ll64_option=64`        | `-mll64`             |
| Unaligned access      | `-unaligned_option`                | `-Xunaligned` | `nsim_isa_unaligned_option=1`    | `-munaligned-access` |
| XY memory             | XY memory option                   | `-Xxy`        | `nsim_isa_xy=1`                  | `-mxy`               |

## Multiplication Options for ARC HS3x/4x

| GCC                           | CCAC/MDB                 | nSIM                        | ARChitect                |
|-------------------------------|--------------------------|-----------------------------|--------------------------|
| `-mmpy-option=0`              | `-Xmpy_option=0`         | `nsim_isa_mpy_option=0`     | `-mpy_option=none`       |
| `-mmpy-option={2,wlh1,mpy}`   | `-Xmpy_option={2-6,mpy}` | `nsim_isa_mpy_option={2-6}` | `-mpy_option=mpy`        |
| `-mmpy-option={7,plus_dmpy}`  | `-Xmpy_option={7,mac}`   | `nsim_isa_mpy_option=7`     | `-mpy_option=plus_dmpy`  |
| `-mmpy-option={8,plus_macd}`  | `-Xmpy_option={8,mpyd}`  | `nsim_isa_mpy_option=8`     | `-mpy_option=plus_macd`  |
| `-mmpy-option={9,plus_qmacw}` | `-Xmpy_option={9,qmpyh}` | `nsim_isa_mpy_option=9`     | `-mpy_option=plus_qmacw` |

## Multiplication Options for ARC EM

| GCC                         | CCAC/MDB                | nSIM                    | ARChitect          |
|-----------------------------|-------------------------|-------------------------|--------------------|
| `-mmpy-option=0`            | `-Xmpy_option=0`        | `nsim_isa_mpy_option=0` | `-mpy_option=none` |
| `-mmpy-option={2,wlh1,mpy}` | `-Xmpy_option={2,wlh1}` | `nsim_isa_mpy_option=2` | `-mpy_option=wlh1` |
| `-mmpy-option={3,wlh2}`     | `-Xmpy_option={3,wlh2}` | `nsim_isa_mpy_option=3` | `-mpy_option=wlh2` |
| `-mmpy-option={4,wlh3}`     | `-Xmpy_option={4,wlh3}` | `nsim_isa_mpy_option=4` | `-mpy_option=wlh3` |
| `-mmpy-option={5,wlh4}`     | `-Xmpy_option={5,wlh4}` | `nsim_isa_mpy_option=5` | `-mpy_option=wlh4` |
| `-mmpy-option={6,wlh5}`     | `-Xmpy_option={6,wlh5}` | `nsim_isa_mpy_option=6` | `-mpy_option=wlh5` |

## Multiplication Options for ARC 700

| Description         | CCAC/MDB | nSIM             | GCC    |
|---------------------|----------|------------------|--------|
| Extended Arithmetic | `-Xea`   | `nsim_isa_sat=1` | `-mea` |

## Multiplication Options for ARC 600

| Description          | CCAC/MDB     | nSIM                  | GCC          |
|----------------------|--------------|-----------------------|--------------|
| 32x16 multiplication | `-Xmul32x16` | `nsim_isa_mul32x16=1` | `-mmul32x16` |
| 32x32 multiplication | `-Xmult32d`  | `nsim_isa_mult32d=1`  | `-mmul64`    |
| Extended Arithmetic  | `-Xea`       | `nsim_isa_sat=1`      | N/A          |

## FPU options for ARC HS3x/4x

| GCC               | Families | ARChitect                                                 | nSIM                                                                           | CCAC/MDB                       |
|-------------------|----------|-----------------------------------------------------------|--------------------------------------------------------------------------------|--------------------------------|
| `-mfpu=fpus`      | EM, HS   | `-has_fpu`                                                | `nsim_isa_fpus_option=1`                                                       | `-Xfpus`                       |
| `-mfpu=fpus_div`  | EM, HS   | `-has_fpu -fpu_div_option`                                | `nsim_isa_fpus_option=1 nsim_isa_fpus_div_option=1`                            | `-Xfpus -Xfpus_div`            |
| `-mfpu=fpus_fma`  | EM, HS   | `-has_fpu -fpu_fma_option`                                | `nsim_isa_fpus_option=1 nsim_isa_fpu_mac_option=1`                             | `-Xfpus -Xfpu_mac`             |
| `-mfpu=fpus_all`  | EM, HS   | `-has_fpu -fpu_div_option -fpu_fma_option`                | `nsim_isa_fpus_option=1 nsim_isa_fpus_div_option=1 nsim_isa_fpu_mac_option=1`  | `-Xfpus -Xfpus_div -Xfpu_mac`  |
| `-mfpu=fpud`      | HS       | `-has_fpu -fpu_dp_option`                                 | `nsim_isa_fpud_option=1`                                                       | `-Xfpud`                       |
| `-mfpu=fpud_div`  | HS       | `-has_fpu -fpu_div_option -fpu_dp_option`                 | `nsim_isa_fpud_option=1 nsim_isa_fpud_div_option=1`                            | `-Xfpud -Xfpud_div`            |
| `-mfpu=fpud_fma`  | HS       | `-has_fpu -fpu_fma_option -fpu_dp_option`                 | `nsim_isa_fpud_option=1 nsim_isa_fpu_mac_option=1`                             | `-Xfpud -Xfpu_mac`             |
| `-mfpu=fpud_all`  | HS       | `-has_fpu -fpu_div_option -fpu_fma_option -fpu_dp_option` | `nsim_isa_fpud_option=1 nsim_isa_fpud_div_option=1 nsim_isa_fpu_mac_option=1`  | `-Xfpud -Xfpud_div -Xfpu_mac`  |
| `-mfpu=fpuda`     | EM       | `-fpu_dp_assist`                                          | `nsim_isa_fpuda_option=1`                                                      | `-Xfpuda`                      |
| `-mfpu=fpuda_div` | EM       | `-fpu_dp_assist -fpu_div_option`                          | `nsim_isa_fpuda_option=1 nsim_isa_fpud_div_option=1`                           | `-Xfpuda -Xfpus_div`           |
| `-mfpu=fpuda_fma` | EM       | `-fpu_dp_assist -fpu_fma_option`                          | `nsim_isa_fpuda_option=1 nsim_isa_fpu_mac_option=1`                            | `-Xfpuda -Xfpu_mac`            |
| `-mfpu=fpuda_all` | EM       | `-fpu_dp_assist -fpu_div_option -fpu_fma_option`          | `nsim_isa_fpuda_option=1 nsim_isa_fpud_div_option=1 nsim_isa_fpu_mac_option=1` | `-Xfpuda -Xfpus_div -Xfpu_mac` |
