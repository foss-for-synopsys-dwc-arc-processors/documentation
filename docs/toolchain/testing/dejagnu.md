# Running DejaGNU Tests

!!! info

    You can run DejaGNU tests using scripts after building the toolchain using the
    [arc-gnu-toolchain](https://github.com/foss-for-synopsys-dwc-arc-processors/arc-gnu-toolchain)
    repository.

## Prerequisites

[`toolchain`](https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain)
repository contains DejaGNU scripts for ARC.
[`newlib`](https://github.com/foss-for-synopsys-dwc-arc-processors/newlib) and
[`gcc`](https://github.com/foss-for-synopsys-dwc-arc-processors/gcc)
repositories contain corresponding tests. Prepare an environment for running
DejaGNU tests in a separate working directory (for example, `/home/user/dejagnu`):

```text
mkdir -p /home/user/dejagnu
cd /home/user/dejagnu
git clone https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain
git clone https://github.com/foss-for-synopsys-dwc-arc-processors/newlib
git clone https://github.com/foss-for-synopsys-dwc-arc-processors/gcc
```

Add paths for ARCv1/ARCv2 and ARCv3 toolchains to `PATH` environment variable:

```shell
export PATH="/tools/arc-elf32/bin:$PATH"
export PATH="/tools/arc64-elf/bin:$PATH"
```

Set `WORKING_DIR` environment variable with path of the working
directory for convenience:

```shell
export WORKING_DIR="/home/user/dejagnu"
```

Set environment variables for QEMU simulator:

```shell
export QEMU_HOME="/tools/qemu"
export PATH="$QEMU_HOME/bin:$PATH"
```

Set environment variables for nSIM simulator:

```shell
export NSIM_HOME="/tools/mwdt/nSIM/nSIM_64"
export PATH="$NSIM_HOME/bin:$PATH"
export SNPSLMD_LICENSE_FILE="<your-license-file>"
```

Export multilib options through `ARC_MULTILIB_OPTIONS` variable
(for example, `cpu=hs38` will be passed to GCC as `-mcpu=hs38`):

```shell
export ARC_MULTILIB_OPTIONS="cpu=hs38"
```

## Constructing `site.exp` File

`site.exp` file is a main configuration file for DejaGNU tests. It consists
several parts. Firstly, choose a toolchain:

<table>
    <tr>
        <th>Toolchain</th>
        <th><code>site.exp</code></th>
    </tr>
    <tr>
        <td style="vertical-align: middle">ARCv1/ARCv2</td>
        <td>
```tcl
set target_triplet arc-unknown-elf32
set target_alias arc-elf32
```
        </td>
    </tr>
    <tr>
        <td style="vertical-align: middle">ARCv3</td>
        <td>
```tcl
set target_triplet arc64-unknown-elf
set target_alias arc64-elf
```
        </td>
    </tr>
</table>

Choose a simulator:

<table>
    <tr>
        <th>Simulator</th>
        <th><code>site.exp</code></th>
    </tr>
    <tr>
        <td style="vertical-align: middle">nSIM</td>
        <td>
```tcl
set target_list arc-sim-nsimdrv
```
        </td>
    </tr>
    <tr>
        <td style="vertical-align: middle">QEMU</td>
        <td>
```tcl
set target_list arc-sim-qemu
```
        </td>
    </tr>
</table>

Choose a tool for testing:

<table>
    <tr>
        <th>Tool</th>
        <th><code>site.exp</code></th>
    </tr>
    <tr>
        <td style="vertical-align: middle">GCC</td>
        <td>
```tcl
set tool gcc
set srcdir "$env(WORKING_DIR)/gcc/gcc/testsuite"
```
        </td>
    </tr>
    <tr>
        <td style="vertical-align: middle">Newlib</td>
        <td>
```tcl
set tool newlib
set tool_version "4.3.0"
set srcdir "$env(WORKING_DIR)/newlib/newlib/testsuite"
```
        </td>
    </tr>
    <tr>
        <td style="vertical-align: middle">C++ standard library</td>
        <td>
```tcl
set tool libstdc++
set baseline_subdir_switch "--print-multi-directory"
set srcdir "$env(WORKING_DIR)/gcc/libstdc++-v3/testsuite"
```
        </td>
    </tr>
</table>

Set paths to DejaGNU scripts for ARC:

```tcl
set boards_dir "$env(WORKING_DIR)/toolchain/dejagnu"
lappend boards_dir "$env(WORKING_DIR)/toolchain/dejagnu/baseboards"
```

Set a temporary directory for output files:

```tcl
set tmpdir "$env(WORKING_DIR)/tmp"
```

Set verbosity from 0 to 9:

```tcl
set verbose 0
```

## Examples

### Running Newlib tests using QEMU for ARCv2 with `-mcpu=hs38`

Content of `site.exp`:

```tcl
set target_triplet arc-unknown-elf32
set target_alias arc-elf32
set target_list arc-sim-qemu

set tool newlib
set tool_version "4.3.0"
set srcdir "$env(WORKING_DIR)/newlib/newlib/testsuite"

set tmpdir "$env(WORKING_DIR)/tmp"
set boards_dir "$env(WORKING_DIR)/toolchain/dejagnu"
lappend boards_dir "$env(WORKING_DIR)/toolchain/dejagnu/baseboards"

set verbose 0
```

Running tests:

```shell
export QEMU_HOME="/tools/qemu"
export PATH="$QEMU_HOME/bin:$PATH"
export ARC_MULTILIB_OPTIONS="cpu=hs38"
export WORKING_DIR=$(dirname "$0")
export PATH="/tools/arc-elf32:$PATH"

mkdir -p $WORKING_DIR/tmp

runtest
```

An example of output:

```text
Using ./newlib/newlib/testsuite/lib/newlib.exp as tool init file.
Test run by ykolerov on Wed Aug  9 14:14:10 2023
Target is arc-unknown-elf32
Host   is x86_64-pc-linux-gnu

                === newlib tests ===

Schedule of variations:
    arc-sim-qemu

Running target arc-sim-qemu
Using ./toolchain/dejagnu/baseboards/arc-sim-qemu.exp as board description file for target.
Using /usr/share/dejagnu/config/sim.exp as generic interface file for target.
Using /usr/share/dejagnu/baseboards/basic-sim.exp as board description file for target.
Using ./newlib/newlib/testsuite/config/default.exp as tool-and-target-specific interface file.
Running ./newlib/newlib/testsuite/newlib.elix/elix.exp ...
Running ./newlib/newlib/testsuite/newlib.iconv/iconv.exp ...
Running ./newlib/newlib/testsuite/newlib.locale/UTF-8.exp ...
FAIL: newlib.locale/UTF-8.c output
Running ./newlib/newlib/testsuite/newlib.locale/locale.exp ...
Running ./newlib/newlib/testsuite/newlib.search/hsearchtest.exp ...
Running ./newlib/newlib/testsuite/newlib.stdio/stdio.exp ...
Running ./newlib/newlib/testsuite/newlib.stdlib/atexit.exp ...
Running ./newlib/newlib/testsuite/newlib.stdlib/stdlib.exp ...
Running ./newlib/newlib/testsuite/newlib.string/string.exp ...
Running ./newlib/newlib/testsuite/newlib.time/time.exp ...
Running ./newlib/newlib/testsuite/newlib.wctype/wctype.exp ...
FAIL: newlib.wctype/twctype.c compilation
FAIL: newlib.wctype/twctrans.c compilation

                === newlib Summary ===

# of expected passes            29
# of unexpected failures        3
# of unresolved testcases       2
```

### Running GCC tests using nSIM for ARCv3 with `-mcpu=hs68`

```tcl
set target_triplet arc64-unknown-elf
set target_alias arc64-elf
set target_list arc-sim-nsimdrv

set tool gcc
set srcdir "$env(WORKING_DIR)/gcc/gcc/testsuite"

set tmpdir "$env(WORKING_DIR)/tmp"
set boards_dir "$env(WORKING_DIR)/toolchain/dejagnu"
lappend boards_dir "$env(WORKING_DIR)/toolchain/dejagnu/baseboards"

set verbose 0
```

Running tests:

```shell
export NSIM_HOME="/tools/mwdt/nSIM/nSIM_64"
export PATH="$NSIM_HOME:$PATH"
export SNPSLMD_LICENSE_FILE="<your-license-file>"
export ARC_MULTILIB_OPTIONS="cpu=hs68"
export WORKING_DIR=$(dirname "$0")
export PATH="/tools/toolchains/arc64-elf:$PATH"

mkdir -p $WORKING_DIR/tmp

runtest
```

## Compatibility Tests

GCC contains a set of [compatibility tests](https://gcc.gnu.org/onlinedocs/gcc/Compatibility.html)
named `compat.exp`. It allows to test compatibility of GCC with different compilers.

If you want to run these tests it is necessary to configure additional variables in site.exp file: 

```tcl
# Enable compatibility tests
set is_gcc_compat_suite "1"

# A different compiler is used (for example, clang-based)
set compat_same_alt "0"

# Flags to be used by both compilers
set TEST_ALWAYS_FLAGS  "$env(TEST_ALWAYS_FLAGS)"

# Path to the alternate compiler used for C/C++
set ALT_CC_UNDER_TEST  "$env(COMPAT_ALT_PATH)"
set ALT_CXX_UNDER_TEST "$env(COMPAT_ALT_PATH)"

# Flags for gcc and alternate compiler respectively
set COMPAT_OPTIONS [list [list "$env(GCC_COMPAT_GCC_OPTIONS)" "$env(GCC_COMPAT_CCAC_OPTIONS)"]]

# Disable tests with packed structures to avoid unaligned access errors
set COMPAT_SKIPS [list {ATTRIBUTE}]
```

Here is an example of `site.exp` file for running compatibility tests for GCC
and MetaWare CCAC compilers on nSIM simulator for ARC EM family:

```tcl
set target_triplet arc-unknown-elf32
set target_alias arc-elf32
set target_list arc-sim-nsimdrv

set tool gcc
set srcdir "$env(WORKING_DIR)/gcc/gcc/testsuite"

set tmpdir "$env(WORKING_DIR)/tmp"
set boards_dir "$env(WORKING_DIR)/toolchain/dejagnu"
lappend boards_dir "$env(WORKING_DIR)/toolchain/dejagnu/baseboards"

set is_gcc_compat_suite "1"
set compat_same_alt "0"
set TEST_ALWAYS_FLAGS  "$env(TEST_ALWAYS_FLAGS)"
set ALT_CC_UNDER_TEST  "$env(COMPAT_ALT_PATH)"
set ALT_CXX_UNDER_TEST "$env(COMPAT_ALT_PATH)"
set COMPAT_OPTIONS [list [list "$env(GCC_COMPAT_GCC_OPTIONS)" "$env(GCC_COMPAT_CCAC_OPTIONS)"]]
set COMPAT_SKIPS [list {ATTRIBUTE}]

set verbose 0
```

Running tests:

```shell
export NSIM_HOME="/tools/mwdt/nSIM/nSIM_64"
export PATH="$NSIM_HOME:$PATH"
export SNPSLMD_LICENSE_FILE="<your-license-file>"
export WORKING_DIR=$(dirname "$0")
export PATH="/tools/toolchains/arc-elf32:$PATH"

export METAWARE_ROOT="/tools/mwdt/MetaWare"
export COMPAT_ALT_PATH="$METAWARE_ROOT/arc/bin/ccac"
export TEST_ALWAYS_FLAGS="-O0 -g"
export GCC_COMPAT_GCC_OPTIONS="-mcpu=em4_dmips -mno-sdata \
        -fshort-enums -Wl,-z,muldefs -Wl,--no-warn-mismatch -lgcc -lnsim -lc \
        -lg -lm -L$METAWARE_ROOT/arc/lib/av2em/le -lmw"
export GCC_COMPAT_CCAC_OPTIONS="-av2em -Xbasecase -Hnocopyr -Hnosdata -fstrict-abi"

mkdir -p $WORKING_DIR/tmp

runtest compat.exp
```

Options for other targets:

```shell
# ARCv2 HS3x/4x
export GCC_COMPAT_GCC_OPTIONS="-mcpu=archs ... -L$METAWARE_ROOT/arc/lib/av2hs/le"
export GCC_COMPAT_CCAC_OPTIONS="-arcv2hs ..."

# ARCv3 HS5x
export GCC_COMPAT_GCC_OPTIONS="-mcpu=hs5x ... -L$METAWARE_ROOT/arc/lib/av3hs/le"
export GCC_COMPAT_CCAC_OPTIONS="-arcv3hs ..."

# ARCv3 HS6x
export GCC_COMPAT_GCC_OPTIONS="-mcpu=hs6x ... -L$METAWARE_ROOT/arc/lib/arc64/le"
export GCC_COMPAT_CCAC_OPTIONS="-arc64 ..."
```
