# Toolchains for ARC Processors

## Toolchains for Baremetal Targets

GNU toolchains for ARC processors consist of GCC, Binutils and GDB. [Newlib](https://sourceware.org/newlib/) standard library is used for building baremetal applications. This table depicts which GCC driver should be used depending on ISA:

| ISA    | Driver/Triplet |
|--------|----------------|
| ARCv1  | `arc-elf32-gcc`|
| ARCv2  | `arc-elf32-gcc`|
| ARCv3  | `arc64-elf-gcc`|

Note that binaries for both ARCv1 and ARCv2 may be built using a single `arc-elf32-gcc` driver. It means
that there is a single toolchain for two ISAs.

## Toolchains for Linux Targets

There is a set of GNU toolchains for ARC processors which allow to build and debug
applications for Linux. The Linux kernel itself for ARC processors may be built using
both baremetal and Linux toolchains.

Linux toolchains are presented for all ARC processor families except ARC EM and ARC 600.
Two Linux standard libraries are currently supported: glibc and uClibc-ng.
This table depicts which GCC driver should be used depending on CPU family:

| Family             | Standard library | Driver/Triplet           |
| -----------------  | ---------------- | ------------------------ |
| ARC 700            | glibc            | `arc-linux-gnu-gcc`      |
| ARC 700            | uClibc-ng        | `arc-linux-uclibc-gcc`   |
| ARC HS3x/4x/4xD    | glibc            | `arc-linux-gnu-gcc`      |
| ARC HS3x/4x/4xD    | uClibc-ng        | `arc-linux-uclibc-gcc`   |
| ARC HS5x           | uClibc-ng        | `arc32-linux-uclibc-gcc` |
| ARC HS6x           | glibc            | `arc64-linux-gnu-gcc`    |

Note, that in case of Linux toolchains, `arc-linux-*` drivers for ARC 700 and ARC HS are
different toolchains! For example, if you want to build applications for ARC 700 Linux targets,
then you need to use a particular toolchain for ARC 700 and the same is applicable for ARC HS,
though they have the same names.

## How to Get The Toolchain

There are several ways of getting the toolchain:

1. Download the latest release of the prebuilt toolchain on [the release page](https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain/releases)
   of main [toolchain's repository](https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain). Also, Eclipse IDE
   is available for downloading which is shipped with some of toolchains and OpenOCD.
2. You can build toolchains using Crosstool-NG build system. Follow instructions presented in
   `README.md` of main [toolchain's repository](https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain)
   on GitHub.
3. You can build some of toolchains using scripts from [arc-gnu-toolchain](https://github.com/foss-for-synopsys-dwc-arc-processors/arc-gnu-toolchain)
   repository. These scripts are based on the [RISC-V scripts](https://github.com/riscv/riscv-gnu-toolchain).
