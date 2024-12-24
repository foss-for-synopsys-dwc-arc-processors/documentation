# GNU Toolchain for ARC-V Targets

## Overview

In `arc-2024.12` release GNU toolchain for ARC-V targets is based on these
tools and libraries:

* GCC 14.2 with ARC-V specific patches
    * Uses upstream 14.2 release; see [a release announcement](https://lists.gnu.org/archive/html/info-gnu/2024-08/msg00000.html)
      and [a complete list of changes](https://gcc.gnu.org/gcc-14/changes.html).
    * Added full support of tuning instructions scheduling for RMX-100, RMX-500 and RHX-100.
* Binutils 2.43 with ARC-V specific patches
    * Uses upstream 2.43 release; see [release notes](https://lists.gnu.org/archive/html/info-gnu/2024-08/msg00001.html).
* GDB 15.1
    * Uses upstream 15.1 release; see [a release announcement](https://lists.gnu.org/archive/html/info-gnu/2024-07/msg00004.html)
      and [a complete list of changes](https://sourceware.org/git/gitweb.cgi?p=binutils-gdb.git;a=blob_plain;f=gdb/NEWS;hb=gdb-15.1-release).
* Newlib 4.4.0 with ARC-V specific patches
    * Uses upstream 4.4.0 release; see [release announcement](https://sourceware.org/pipermail/newlib/2023/020873.html).
    * Caches are enabled automatically when a custom startup file is used.
    * Added support of passing command line arguments through Semihosting interface (works both for nSIM and QEMU simulators).

Supported ARC-V specific features:

* [Tuning GCC instruction scheduling](#tuning-scheduling) for RMX-100, RMX-500 and RHX-100 targets.
* Includes optimized sets of libraries for basic profiles for RMX-100, RMX-500 and RHX-100.
* Support of enabling ARC-V caches on startup.

Because the ARC-V GNU toolchain is built on top of standard components such as GCC, Binutils, and GDB,
all functionality of these tools stays in place and can be studied in detail in the corresponding manuals:

* [GCC documentation](https://gcc.gnu.org/onlinedocs/14.2.0/)
* [Binutils documentation](https://sourceware.org/binutils/docs-2.43/)
* [GDB documentation](https://www.sourceware.org/gdb/documentation/)

## Known issues

* The size-optimized Newlib Nano configuration (used when `-specs=nano.specs` is passed to GCC) does not
  support `printf()` for `float` and `double` types by default. If you need that feature, pass `-u _printf_float`
  to GCC when you compile your applications. This option picks up support of `float` and `double` for size
  optimized `printf()` on demand.
* Some complex combinations of `-march=` and `-mabi=` options might lead to unpredictable errors during
  compilation. Such issues are related to general support of RISC-V extensions in GCC.
  Refer [#610](https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain/issues/610) for details.
* There is an issue in GCC that prevents matching `-march` configurations like `rv32imc` and `rv32im_zca`
  as compatible and interchangeable. Refer [#653](https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain/issues/653)
  for details.
