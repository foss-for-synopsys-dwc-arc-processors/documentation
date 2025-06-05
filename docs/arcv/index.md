# GNU Toolchain for ARC-V Targets

## Overview

Supported ARC-V specific features:

* [Tuning GCC instruction scheduling](#tuning-scheduling) for RMX-100, RMX-500, RHX-100 and RPX-100 targets.
* Includes optimized sets of libraries for basic profiles for RMX-100, RMX-500, RHX-100 and RPX-100.
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
* Using selective scheduler with a specific set of options may lead to incorrect code generation. Refer
  [Bug 118153](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=118153) for details.
