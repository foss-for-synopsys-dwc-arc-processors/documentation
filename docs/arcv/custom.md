# Building Toolchains

The GNU toolchain for ARC-V targets contains a set of precompiled standard libraries
for different combinations of `-march`, `-mabi` and `-mcmodel`. You can find a list
of available configurations this way (they are described in
[Understanding ARC-V Configurations](./building.md#understanding-arc-v-configurations) section):

```
$ riscv64-snps-elf-gcc -print-multi-lib
.;
rv32e/ilp32e/rmx100;@march=rv32e@mabi=ilp32e@mtune=arc-v-rmx-100-series
rv32i/ilp32;@march=rv32i@mabi=ilp32
rv32i/ilp32/rmx100;@march=rv32i@mabi=ilp32@mtune=arc-v-rmx-100-series
rv32i/ilp32/rmx500;@march=rv32i@mabi=ilp32@mtune=arc-v-rmx-500-series
rv32i/ilp32/rhx100;@march=rv32i@mabi=ilp32@mtune=arc-v-rhx-100-series
rv32ia/ilp32/rmx100;@march=rv32ia@mabi=ilp32@mtune=arc-v-rmx-100-series
rv32ia/ilp32/rmx500;@march=rv32ia@mabi=ilp32@mtune=arc-v-rmx-500-series
rv32ia/ilp32/rhx100;@march=rv32ia@mabi=ilp32@mtune=arc-v-rhx-100-series
rv32iac_zicsr/ilp32/rmx100;@march=rv32iac_zicsr@mabi=ilp32@mtune=arc-v-rmx-100-series
rv32iac_zicsr/ilp32/rmx500;@march=rv32iac_zicsr@mabi=ilp32@mtune=arc-v-rmx-500-series
rv32imc_zicsr/ilp32/rmx100;@march=rv32imc_zicsr@mabi=ilp32@mtune=arc-v-rmx-100-series
rv32imc_zicsr/ilp32/rmx500;@march=rv32imc_zicsr@mabi=ilp32@mtune=arc-v-rmx-500-series
rv32imac_zicsr/ilp32/rmx100;@march=rv32imac_zicsr@mabi=ilp32@mtune=arc-v-rmx-100-series
rv32imac_zicsr/ilp32/rmx500;@march=rv32imac_zicsr@mabi=ilp32@mtune=arc-v-rmx-500-series
rv32imac_zicsr/ilp32/rhx100;@march=rv32imac_zicsr@mabi=ilp32@mtune=arc-v-rhx-100-series
rv32imafc/ilp32f/rhx100;@march=rv32imafc@mabi=ilp32f@mtune=arc-v-rhx-100-series
rv32imafdc/ilp32d/rhx100;@march=rv32imafdc@mabi=ilp32d@mtune=arc-v-rhx-100-series
rv64i/lp64/rmx100/medany;@march=rv64i@mabi=lp64@mtune=arc-v-rmx-100-series@mcmodel=medany
rv64ia/lp64/rmx100/medany;@march=rv64ia@mabi=lp64@mtune=arc-v-rmx-100-series@mcmodel=medany
rv64imac_zicsr/lp64/rmx100/medany;@march=rv64imac_zicsr@mabi=lp64@mtune=arc-v-rmx-100-series@mcmodel=medany
rv64imafdc/lp64d/rmx100/medany;@march=rv64imafdc@mabi=lp64d@mtune=arc-v-rmx-100-series@mcmodel=medany
rv32emac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zdinx_zicsr/ilp32e/rmx100;@march=rv32emac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zdinx_zicsr@mabi=ilp32e@mtune=arc-v-rmx-100-series
rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr/ilp32/rmx100;@march=rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr@mabi=ilp32@mtune=arc-v-rmx-100-series
rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr/ilp32/rmx500;@march=rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr@mabi=ilp32@mtune=arc-v-rmx-500-series
rv32im_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr/ilp32/rmx100;@march=rv32im_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr@mabi=ilp32@mtune=arc-v-rmx-100-series
rv32im_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr/ilp32/rmx500;@march=rv32im_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr@mabi=ilp32@mtune=arc-v-rmx-500-series
rv32imc_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr/ilp32/rmx100;@march=rv32imc_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr@mabi=ilp32@mtune=arc-v-rmx-100-series
rv32imc_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr/ilp32/rmx500;@march=rv32imc_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr@mabi=ilp32@mtune=arc-v-rmx-500-series
rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr/ilp32/rmx100;@march=rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr@mabi=ilp32@mtune=arc-v-rmx-100-series
rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr/ilp32/rmx500;@march=rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr@mabi=ilp32@mtune=arc-v-rmx-500-series
rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zicsr/ilp32/rmx100;@march=rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zicsr@mabi=ilp32@mtune=arc-v-rmx-100-series
rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zicsr/ilp32/rmx500;@march=rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zicsr@mabi=ilp32@mtune=arc-v-rmx-500-series
rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zdinx_zicsr/ilp32/rmx100;@march=rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zdinx_zicsr@mabi=ilp32@mtune=arc-v-rmx-100-series
rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zdinx_zicsr/ilp32/rmx500;@march=rv32imac_zcb_zcmp_zcmt_zba_zbb_zbs_zfinx_zdinx_zicsr@mabi=ilp32@mtune=arc-v-rmx-500-series
rv32imac_zcb_zcmp_zba_zbb_zbs_zicsr/ilp32/rhx100;@march=rv32imac_zcb_zcmp_zba_zbb_zbs_zicsr@mabi=ilp32@mtune=arc-v-rhx-100-series
rv32imafc_zcb_zcmp_zba_zbb_zbs_zicsr/ilp32f/rhx100;@march=rv32imafc_zcb_zcmp_zba_zbb_zbs_zicsr@mabi=ilp32f@mtune=arc-v-rhx-100-series
rv32imafd_zca_zcb_zcmp_zba_zbb_zbs_zicsr/ilp32d/rhx100;@march=rv32imafd_zca_zcb_zcmp_zba_zbb_zbs_zicsr@mabi=ilp32d@mtune=arc-v-rhx-100-series
rv64imac_zcb_zba_zbb_zbs_zicsr/lp64/rmx100/medany;@march=rv64imac_zcb_zba_zbb_zbs_zicsr@mabi=lp64@mtune=arc-v-rmx-100-series@mcmodel=medany
rv64imafdc_zcb_zba_zbb_zbs_zicsr/lp64d/rmx100/medany;@march=rv64imafdc_zcb_zba_zbb_zbs_zicsr@mabi=lp64d@mtune=arc-v-rmx-100-series@mcmodel=medany
```

However, sometimes a particular configuration is needed which is not shipped with a prebuilt toolchain.
To achieve this it's necessary to build GNU toolchain from scratch. The easiest way to build a custom toolchain
for another custom configuration is to use Crosstool-NG. You can find a comprehensive guide about how to get and prepare
Crosstool-NG here: https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain.

Checkout and build Crosstool-NG:

```
$ git clone -b arc-2024.12 https://github.com/foss-for-synopsys-dwc-arc-processors/crosstool-ng
$ ./bootstrap
$ ./configure --enable-local
$ make
```

Select a default ARC-V configuration file and enter a configuration menu:

```
$ ./ct-ng snps-riscv64-unknown-elf
$ ./ct-ng menuconfig
```

For example, if you want to build Crosstool-NG for
`-march=rv32imafc_zicond_zicsr_zifencei -mabi=ilp32f -mtune=arc-v-rmx-100-series`
then do these steps:

1. Unselect `Target Options -> Build a multilib toolchain`.
2. Set `Target Options -> Architecture Level` to `rv32imafc_zicond_zicsr_zifencei`.
3. Set `Target Options -> Generate code for the specific ABI` to `ilp32f`.
4. Set `Target Options -> Tune for CPU` to `-mtune=arc-v-rmx-100-series`.

Then build the toolchain as usual:

```
$ ./ct-ng build
```

The toolchain may be found in `riscv64-unknown-elf` directory. It uses
`-march=rv32imafc_zicond_zicsr_zifencei -mabi=ilp32f -mtune=arc-v-rmx-100-series`
options by default.
