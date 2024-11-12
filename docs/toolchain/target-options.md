# Target Specific Options

## Overview

In addition GCC options common to all targets, ARC GNU toolchains supports a set of `-m<value>` target specific options.
Each of them configures a particular feature. For example, `-mno-code-density` option disables generating
code density instructions.

`-mcpu=<core>` selects a particular ISA and CPU family and enables/disables a set of other `-m` options.
Each `-mcpu` value leads to linking with a prebuilt library which corresponds to this particular
`-mcpu=<value>` and is built using this `-mcpu=<value>`.

For example, options combination `-mcpu=em4 -mno-code-density` generates code that doesn't
use code density instructions, however it is linked with standard library that has been built
with just `-mcpu=em4`, which uses code density instructions. Therefore a final application still may
use code density instructions.

All available target specific options may be found in an official [GCC manual][gcc-manual].

## ARCv1 and ARCv2 Toolchains

### Target Options

Here is a table of target options for ARCv1 and ARCv2 CPU families
with default values for CPU families (On, Off or N/A).

| Options              | ARC HS | ARC EM | ARC 700 | ARC 600 |
|----------------------|--------|--------|---------|---------|
| `-mbarrel-shifter`   | On     | Off    | On      | Off     |
| `-mnorm`             | On     | Off    | On      | Off     |
| `-mswap`             | On     | Off    | On      | Off     |
| `-rf16`              | Off    | Off    | Off     | Off     |
| `-mfpu=<...>`        | Off    | Off    | —       | —       |
| `-mmpy-option=<...>` | Off    | Off    | —       | —       |
| `-mcode-density`     | On     | Off    | —       | —       |
| `-mdiv-rem`          | Off    | Off    | —       | —       |
| `-matomic`           | On     | —      | Off     | —       |
| `-mll64`             | Off    | —      | —       | —       |
| `-mspfp`             | —      | Off    | Off     | Off     |
| `-mdpfp`             | —      | Off    | Off     | Off     |
| `-msimd`             | —      | Off    | Off     | —       |
| `-mmargonaut`        | —      | —      | Off     | Off     |
| `-mea`               | —      | —      | Off     | —       |
| `-mmul64`            | —      | —      | —       | Off     |
| `-mmul32x16`         | —      | —      | —       | Off     |
| `-munaligned-access` |        |        |         |         |

You can find a short description for all target options using `--target-help` option. For example:

```
$ arc-elf32-gcc --target-help
...
  -mbranch-index              Enable use of BI/BIH instructions when available.
  -mcase-vector-pcrel         Use pc-relative switch case tables - this enables case table shortening.
  -mcmem                      Enable use of NPS400 xld/xst extension.
  -mcode-density              Enable code density instructions for ARCv2.
  -mcode-density-frame        Enable ENTER_S and LEAVE_S opcodes for ARCv2.
  -mcompact-casesi            Enable compact casesi pattern.  Uses of this option are diagnosed.
  -mcpu=CPU                   Compile code for ARC variant CPU.
  -mcrc                       Enable variable polynomial CRC extension.  Uses of this option are diagnosed.
  -mdiv-rem                   Enable DIV-REM instructions for ARCv2.
  -mdpfp                      FPX: Generate Double Precision FPX (compact) instructions.
  -mdpfp-compact              FPX: Generate Double Precision FPX (compact) instructions.
...
```

You can find what target options are enabled or disabled by a particular `-mcpu=<...>` using
`-Q` options. For example:

```
$ arc-elf32-gcc -Q --target-help -mcpu=archs
...
  -mbranch-index                        [disabled]
  -mcase-vector-pcrel                   [disabled]
  -mcmem                                [disabled]
  -mcode-density                        [enabled]
  -mcode-density-frame                  [disabled]
  -mcompact-casesi                      [disabled]
  -mcpu=CPU                             archs
  -mcrc                                 [disabled]
  -mdiv-rem                             [enabled]
  -mdpfp                                [disabled]
  -mdpfp-compact                        [disabled]
...
```

Also, there are several configuration files in GCC's source tree that describe
target options for all targets. All of them may be found in `gcc/config/arc`
directory:

* [`arc-options.def`](https://github.com/foss-for-synopsys-dwc-arc-processors/gcc/blob/arc64/gcc/config/arc/arc-options.def),
[`arc-opts.h`](https://github.com/foss-for-synopsys-dwc-arc-processors/gcc/blob/arc64/gcc/config/arc/arc-opts.h),
[`arc.opt`](https://github.com/foss-for-synopsys-dwc-arc-processors/gcc/blob/arc64/gcc/config/arc/arc.opt) - 
definitions and descriptions for all target options.
* [`arc-arches.def`](https://github.com/foss-for-synopsys-dwc-arc-processors/gcc/blob/arc64/gcc/config/arc/arc-arches.def) -
definitions of CPU families with all available and default target options.
* [`arc-cpus.def`](https://github.com/foss-for-synopsys-dwc-arc-processors/gcc/blob/arc64/gcc/config/arc/arc-cpus.def) -
definitions of particular `-mcpu=<core>` targets.

### Values of `-mcpu` for ARC HS3x and HS4x Families

| `-mcpu=`     | `-mdiv-rem` | `-matomic` | `-mll64` | `-mmpy-option=` | `-mfpu=`   |
|--------------|-------------|------------|----------|-----------------|------------|
| `hs`         |             | Y          |          |                 |            |
| `hs34`       |             | Y          |          | `mpy`           |            |
| `archs`      | Y           | Y          | Y        | `mpy`           |            |
| `hs38`       | Y           | Y          | Y        | `plus_qmacw`    |            |
| `hs4x`       | Y           | Y          | Y        | `plus_qmacw`    |            |
| `hs4xd`      | Y           | Y          | Y        | `plus_qmacw`    |            |
| `hs38_linux` | Y           | Y          | Y        | `plus_qmacw`    | `fpud_all` |

The above `-mcpu` values correspond to specific ARC EM Processor templates presented in the ARChitect tool.

* `-mcpu=hs` corresponds to a basic ARC HS with only atomic instructions enabled. It corresponds to
the following ARC HS templates in ARChitect: `hs34_base`, `hs36_base` and `hs38_base`.
* `-mcpu=hs34` is like `hs` but with with additional support for standard hardware multiplier.
It corresponds to the following ARC HS templates in ARChitect: `hs34`, `hs36` and `hs38`.
* `-mcpu=archs` is a generic configuration, which corresponds to the default configuration in
older GNU toolchain versions.
* `-mcpu=hs38` is a full-featured ARC HS. It corresponds to the following ARC HS template in ARChitect: `hs38_full`.
* `-mcpu=hs4x` and `-mcpu=hs4xd` have the same option set as `-mcpu=hs38`, but compiler will optimize
instruction scheduling for specified processors.
* `-mcpu=hs38_linux` is a full-featured ARC HS with additional support of double-precision FPU.

### Values of `-mcpu` for ARC EM Family

| `-mcpu`      | `-mcode-density` | `-mnorm` | `-mswap` | `-mbarrel-shifter` | `-mdiv-rem` | `-mmpy-option=` | `-mfpu=` | `-mrf16` | `-mspfp` | `-mdpfp` |
|--------------|------------------|----------|----------|--------------------|-------------|-----------------|----------|----------|----------|----------|
| `em`         |                  |          |          |                    |             |                 |          |          |          |          |
| `em_mini`    |                  |          |          |                    |             |                 |          | Y        |          |          |
| `em4`        | Y                |          |          |                    |             |                 |          |          |          |          |
| `arcem`      | Y                |          |          | Y                  |             | `wlh1`          |          |          |          |          |
| `em4_dmips`  | Y                | Y        | Y        | Y                  | Y           | `wlh1`          |          |          |          |          |
| `em4_fpus`   | Y                | Y        | Y        | Y                  | Y           | `wlh1`          | `fpus`   |          |          |          |
| `em4_fpuda`  | Y                | Y        | Y        | Y                  | Y           | `wlh1`          | `fpuda`  |          |          |          |
| `quarkse_em` | Y                | Y        | Y        | Y                  | Y           | `wlh2`          | `quark`  |          | Y        | Y        |

The above `-mcpu` values correspond to specific ARC EM Processor templates presented in the ARChitect tool.
It should be noted however that some ARC features are not currently supported in the GNU toolchain, for
example DSP instruction support.

* `-mcpu=em` doesn't correspond to any specific template, it simply defines the base ARC EM configuration without any optional instructions.
* `-mcpu=em_mini` is same as `em`, but uses reduced register file with only 16 core registers.
* `-mcpu=em4` is a base ARC EM core configuration with `-mcode-density` option. It corresponds to
the following ARC EM templates in ARChitect: `em4_mini`, `em4_sensor`, `em4_ecc`, `em6_mini`, `em5d_mini`,
`em5d_mini_v3`, `em5d_nrg`, `em7d_nrg`, `em9d_mini`. Note that those mini templates have a reduced core register
file, but `-mcpu=em4` doesn't enable this feature.
* `-mcpu=arcem` doesn't correspond to any specific template, it is a legacy flag preserved for compatibility with
older GNU toolchain versions, where `-mcpu` used to select only a CPU family, while optional features were enabled
or disabled by individual `-m<something>` options.
* `-mcpu=em4_dmips` is a full-featured ARC EM configuration for integer operations. It corresponds to the following
ARC EM templates in ARChitect: `em4_dmips`, `em4_rtos`, `em6_dmips`, `em4_dmips_v3`, `em4_parity`, `em6_dmips_v3`,
`em6_gp`, `em5d_voice_audio`, `em5d_nrg_v3`, `em7d_nrg_v3`, `em7d_voice_audio`, `em9d_nrg`, `em9d_voice_audio`,
`em11d_nrg` and `em11d_voice_audio`.
* `-mcpu=em4_fpus` is like `em4_dmips` but with additional support for single-precision floating point unit.
It corresponds to the following ARC EM templates in ARChitect: `em4_dmips_fpusp`, `em4_dmips_fpusp_v3`, `em5d_nrg_fpusp`
and `em9d_nrg_fpusp`.
* `-mcpu=em4_fpuda` is like `em4_fpus` but with additional support for double-precision assist instructions.
It corresponds to the following ARC EM templates in ARChitect: `em4_dmips_fpuspdp` and `em4_dmips_fpuspdp_v3`.
* `-mcpu=quarkse_em` is a configuration for ARC processor in Intel Quark SE chip. It enables additional floating
point instructions which cannot be enable using a particular target option.

### Values of `-mcpu` for ARC 600 and 700 Families

| `-mcpu=`          | `-mnorm` | `-mswap` | `-mbarrel-shifter` | Multiplier   |
|-------------------|----------|----------|--------------------|--------------|
| `arc700`          | Y        | Y        | Y                  | `-mmpy`      |
| `arc600`          |          |          | Y                  |              |
| `arc600_norm`     | Y        |          | Y                  |              |
| `arc600_mul64`    | Y        |          | Y                  | `-mmul64`    |
| `arc600_mul32x16` | Y        |          | Y                  | `-mmul32x16` |
| `arc601`          |          |          |                    |              |
| `arc601_norm`     | Y        |          |                    |              |
| `arc601_mul64`    | Y        |          |                    | `-mmul64`    |
| `arc601_mul32x16` | Y        |          |                    | `-mmul32x16` |

### Controlling the Memory Model

| `-mcmodel` | Memory size for Data/Code     |
|------------|-------------------------------|
| `small`    | 1MB region                    |
| `medium`   | 4GB region (used by `-fpic`)  |
| `large`    | Full memory (used by `-fPIC`) |

### Other Tweaking Options

| Option    | Description                                                              |
|-----------|--------------------------------------------------------------------------|
| `-mfpmov` | Reduces the pressure on GPRs by using FPRs for inline memory operations. |
| `-mbrcc`  | Generate BRcc instructions early on.                                     |
| `-mbbit`  | Likewise but for BBITx instructions.                                     |

## ARCv3 Toolchain

### Values of `-mcpu` for ARCv3

A specific ARCv3 family is selected by `-mcpu=` option. This option may be passed
with these values: `hs5x`, `hs58`, `hs6x` and `hs68`. `hs5x` and `hs58`
stand for ARC HS5x targets and `-mcpu=hs6x` or `-mcpu=hs68` stand for ARC HS6x
targets. Each `-mcpu=` value selects a corresponding prebuilt library for
linking with an application. Here is a list of `-mcpu=` values and corresponding
default values selected for target options:

| Options              | `-mcpu=hs5x` | `-mcpu=hs58` | `-mcpu=hs6x` | `-mcpu=hs68` |
|----------------------|--------------|--------------|--------------|--------------|
| `-m128`              | —            | —            | Off          | On           |
| `-mll64`             | Off          | On           | —            | —            |
| `-matomic=...`       | `1`          | `1`          | `1`          | `1`          |
| `-mbitscan`          | On           | On           | On           | On           |
| `-mcode-density`     | On           | On           | On           | On           |
| `-msimd`             | On           | On           | On           | On           |
| `-mdiv-rem`          | On           | On           | On           | On           |
| `-munaligned-access` | On           | On           | On           | On           |
| `-mbbit`             | Off          | Off          | Off          | Off          |
| `-mbrcc`             | Off          | Off          | Off          | Off          |
| `-mfpu=...`          | `none`       | `none`       | `none`       | `none`       |
| `-mwide`             | On           | On           | On           | On           |
| `-mfpmov`            | Off          | Off          | Off          | Off          |
| `-mvolatile-di`      | Off          | Off          | Off          | Off          |

### General Target Options for ARCv3

Here is a table of target options for ARCv3 CPU families.

| Feature                                                          | GCC                  | CCAC                 | ARChitect                  | nSIM                                  |
|------------------------------------------------------------------|----------------------|----------------------|----------------------------|---------------------------------------|
| Enable 128-bit load and store operations (HS6x only)             | `-m128`              | `-Xm128`             | `-m128_option`             | `-p nsim_isa_m128_option=1`           |
| Enable 64-bit load and store operations (HS5x only)              | `-mll64`             | `-Xll64`             | `-ll64_option`             | `-p nsim_isa_ll64_option=1`           |
| Enable atomic instructions                                       | `-matomic={0,1,2,3}` | `-Xatomic={0,1,2,3}` | `-atomic_option={0,1,2,3}` | `-p nsim_isa_atomic_option={0,1,2,3}` |
| Enable `NORM`, `NORMH`, `FFS`, `FLS`, `NORML`, `FFSL` and `FLSL` | `-mbitscan`          | —                    | —                          | —                                     |
| Enable code-density instructions                                 | `-mcode-density`     | —                    | —                          | —                                     |
| Enable integer SIMD instructions                                 | `-msimd`             | `-Xmpy_option=qmpyh` | —                          | `-p nsim_isa_mpy_option=9`            |
| Enable `DIV` and `REM` instructions                              | `-mdiv-rem`          | `-Xdiv_rem=radix4`   | —                          | `-p nsim_isa_div_rem_option=2`        |
| Enable unaligned access for packed data                          | `-munaligned-access` | `-Xunaligned`        | —                          | `-p nsim_isa_unaligned_option=1`      |
| Generate `BBITx` instructions during combiner step               | `-mbbit`             | —                    | —                          | —                                     |
| Generate `BRcc` instructions during combiner step                | `-mbrcc`             | —                    | —                          | —                                     |
| Enable uncached access for volatile memories                     | `-mvolatile-di`      | —                    | —                          | —                                     |

### FPU Target Options for ARCv3

`-mfpu=...` option enables a set of FPU features. When `-mfpu=none` is passed then FPU
is turned off.

Here is a list of single-presicion FPU features enabled by `-mfpu=fpus`:

| Feature                       | CCAC       | ARChitect        | nSIM                          |
|-------------------------------|------------|------------------|-------------------------------|
| Single-precision instructions | `-Xfp_sp`  | `-has_fp`        | `-p nsim_isa_has_fp=1`        |
| Half-precision instructions   | `-Xfp_hp`  | `-fp_hp_option`  | `-p nsim_isa_fp_hp_option=1`  |
| Division instructions         | `-Xfp_div` | `-fp_div_option` | `-p nsim_isa_fp_div_option=1` |
| Vector instructions           | `-Xfp_vec` | `-fp_vec_option` | `-p nsim_isa_fp_vec_option=1` |

Here is a list of double-presicion FPU features enabled by `-mfpu=fpud`:

| Feature                       | CCAC       | ARChitect        | nSIM                          |
|-------------------------------|------------|------------------|-------------------------------|
| Single-precision instructions | `-Xfp_sp`  | `-has_fp`        | `-p nsim_isa_has_fp=1`        |
| Double-precision instructions | `-Xfp_dp`  | `-fp_dp_option`  | `-p nsim_isa_fp_dp_option=1`  |
| Half-precision instructions   | `-Xfp_hp`  | `-fp_hp_option`  | `-p nsim_isa_fp_hp_option=1`  |
| Division instructions         | `-Xfp_div` | `-fp_div_option` | `-p nsim_isa_fp_div_option=1` |
| Vector instructions           | `-Xfp_vec` | `-fp_vec_option` | `-p nsim_isa_fp_vec_option=1` |

Other FPU options:

| Feature                                                   | GCC       | CCAC        | ARChitect         | nSIM                           |
|-----------------------------------------------------------|-----------|-------------|-------------------|--------------------------------|
| Enable wide floating point SIMD support                   | `-mwide`  | `-Xfp_wide` | `-fp_wide_option` | `-p nsim_isa_fp_wide_option=1` |
| Use FPRs for memory operations like `memcpy` to free GPRs | `-mfpmov` | —           | —                 | —                              |

[gcc-manual]: <https://gcc.gnu.org/onlinedocs/gcc/ARC-Options.html> "GCC manual"
