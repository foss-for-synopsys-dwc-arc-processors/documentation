# Building Toolchains

The GNU toolchain for ARC-V targets contains a set of precompiled standard libraries
for different combinations of `-march`, `-mabi` and `-mcmodel`. You can find a list
of available configurations this way (they are described in
[Understanding ARC-V configurations](./multilib.md) section):

```
$ riscv64-snps-elf-gcc -print-multi-lib
.;
rv32e/ilp32e;@march=rv32e@mabi=ilp32e
rv32em/ilp32e;@march=rv32em@mabi=ilp32e
rv32ea/ilp32e;@march=rv32ea@mabi=ilp32e
rv32ema/ilp32e;@march=rv32ema@mabi=ilp32e
rv32emac_zcb_zba_zbb_zbs_zfinx_zdinx/ilp32e;@march=rv32emac_zcb_zba_zbb_zbs_zfinx_zdinx@mabi=ilp32e
rv32i/ilp32;@march=rv32i@mabi=ilp32
rv32i/ilp32/rmx500;@march=rv32i@mabi=ilp32@mtune=arc-v-rmx-500-series
rv32i/ilp32/rhx100;@march=rv32i@mabi=ilp32@mtune=arc-v-rhx-100-series
rv32ic/ilp32;@march=rv32ic@mabi=ilp32
rv32ic/ilp32/rmx500;@march=rv32ic@mabi=ilp32@mtune=arc-v-rmx-500-series
rv32ic/ilp32/rhx100;@march=rv32ic@mabi=ilp32@mtune=arc-v-rhx-100-series
rv32im/ilp32;@march=rv32im@mabi=ilp32
...
```

However, sometimes a particular configuration is needed which is not shipped with a prebuilt toolchain.
To achieve this it's necessary to build GNU toolchain from scratch. The easiest way to build a custom toolchain
for another custom configuration is to use Crosstool-NG. You can find a comprehensive guide about how to get and prepare
Crosstool-NG here: https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain.

Checkout and build Crosstool-NG:

```
$ git clone -b arc-2025.09 https://github.com/foss-for-synopsys-dwc-arc-processors/crosstool-ng
$ ./bootstrap
$ ./configure --enable-local
$ make
```

Select ARC-V configuration file for Newlib-based toolchain:

```
$ ./ct-ng riscv64-snps-elf-newlib
```

Or select ARC-V configuration file for Picolibc-based toolchain:

```
$ ./ct-ng riscv64-snps-elf-picolibc
```

Enter a configuration menu:

```
$ ./ct-ng menuconfig
```

For example, if you want to build a toolchain only for
`-march=rv32imafc_zicond_zicsr_zifencei -mabi=ilp32f -mtune=arc-v-rmx-100-series`
then do these steps:

1. Unselect `Target Options -> Build a multilib toolchain`.
2. Set `Target Options -> Architecture Level` to `rv32imafc_zicond_zicsr_zifencei`.
3. Set `Target Options -> Generate code for the specific ABI` to `ilp32f`.
4. Set `Target Options -> Tune for CPU` to `-mtune=arc-v-rmx-100-series`.

If you want to set extra GCC options for target libraries you can do this
through `Target Options -> Target CFLAGS`. For example, you can add
`-mstrict-align` to make all target code to be aligned.

Then build the toolchain as usual:

```
$ ./ct-ng build
```

The toolchain may be found in `riscv64-snps-elf-newlib` or 
`riscv64-snps-elf-picolibc` directory. It uses
`-march=rv32imafc_zicond_zicsr_zifencei -mabi=ilp32f -mtune=arc-v-rmx-100-series`
options by default.
