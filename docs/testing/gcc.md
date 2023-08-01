# Running GCC Tests

After building the toolchain using the
[arc-gnu-toolchain](https://github.com/foss-for-synopsys-dwc-arc-processors/arc-gnu-toolchain)
repository, all tests are automatically available as per
[this section in the readme](https://github.com/foss-for-synopsys-dwc-arc-processors/arc-gnu-toolchain#testing-gccc-binutils-and-newlib).

## Standalone testing

It might be useful to perform tests without having to build the entire toolchain.

The gcc tests are based on [dejagnu](https://www.gnu.org/software/dejagnu/) so it
is expected that the framework is installed in your system.

Depending on the simulator used (NSIM or [QEMU](https://github.com/foss-for-synopsys-dwc-arc-processors/qemu))
it is expected that the correct binaries are present in the PATH.

The most direct (manual) way to run the tests is to setup the local environment, include the
necessary sources and binaries, and run using a site.exp file.
A simple test procedure goes as follows:

```shell
$ mkdir test_env; cd test_env           # There are output binaries and logs, so its' useful to keep them separate
$ mkdir tmp                             # Directory used to store the generated objects
$ export TOOLCHAIN_SRC=/path/to/parent  # Path to parent folder of all ARC tool sources (at least gcc, toolchain and arc-gnu-toolchain)
$ export TOOLCHAIN_INST=/path/to/bins   # Path to installed binaries (only required if not in path already)
$ export QEMU_CPU=archs                 # CPU to use in simulation
$ export PATH=$PATH:$TOOLCHAIN_INST/bin:$TOOLCHAIN_SRC/arc-gnu-toolchain/scripts/wrapper/qemu # Add tools and QEMU wrapper to path (used by arc-sim board)
$ cat > site.exp << 'EOF'
set tool gcc

set tmpdir "./tmp"
set srcdir "$env(TOOLCHAIN_SRC)/gcc/gcc/testsuite"

set target_triplet arc-unknown-elf32
set target_alias arc-elf32

set target_list arc-sim

set CFLAGS_FOR_TARGET "--specs=nsim.specs"

set arc_board_dir "$env(TOOLCHAIN_SRC)/toolchain"

if ![info exists boards_dir] {
    lappend boards_dir "$arc_board_dir/dejagnu"
    lappend boards_dir "$arc_board_dir/dejagnu/baseboards"
} else {
    set boards_dir "$arc_board_dir/dejagnu"
    lappend boards_dir "$arc_board_dir/dejagnu/baseboards"
}
EOF
$ runtest        # Now we run the tests!
```

## Other testsuites

GCC has other testsuites that can be enabled by adding specific configurations to site.exp

### Compatibility testsuite

https://gcc.gnu.org/onlinedocs/gcc/Compatibility.html
GCC contains a set of compatibility tests named `compat.exp`. It allows to test compatibility of GCC with different compilers.

If you want to run these tests it is necessary to configure additional variables in `site.exp` file:
```shell
# Enable compat testing
set is_gcc_compat_suite "1"
# Use GCC as the alternate compiler (sanity checking)
set compat_same_alt "1"/"0"

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

Afterwards, launch runtest with `compat.exp` as an argument as well!

### GCC and CCAC compatibility

Some configurations that have been used to test compatibility between GCC and CCAC:

```shell
# Common global flags
export TEST_ALWAYS_FLAGS="-O0 -g"
# Common GCC flags
export GCC_COMPAT_GCC_OPTIONS="-Wl,-z,muldefs -Wl,--no-warn-mismatch -lgcc -lnsim -lg -lm -lmw"
# Common CCAC flags
export GCC_COMPAT_CCAC_OPTIONS="-Xbasecase -Hnocopyr -Hnosdata -fstrict-abi"
```

#### hs6x

```shell
export GCC_COMPAT_GCC_OPTIONS="$GCC_COMPAT_GCC_OPTIONS -mcpu=hs6x -L$METAWARE_ROOT/arc/lib/av2em/le"
export GCC_COMPAT_CCAC_OPTIONS="$GCC_COMPAT_CCAC_OPTIONS -arc64"
```

#### hs5x

```shell
export GCC_COMPAT_GCC_OPTIONS="$GCC_COMPAT_GCC_OPTIONS -mcpu=hs5x -L$METAWARE_ROOT/arc/lib/av3hs/le"
export GCC_COMPAT_CCAC_OPTIONS="$GCC_COMPAT_CCAC_OPTIONS -arcv3hs"
```

#### ARCv2

```shell
export GCC_COMPAT_GCC_OPTIONS="$GCC_COMPAT_GCC_OPTIONS -mcpu=em4_dmips -mno-sdata -fshort-enums -L$METAWARE_ROOT/arc/lib/av2em/le"  
export GCC_COMPAT_CCAC_OPTIONS="$GCC_COMPAT_CCAC_OPTIONS -av2em"
```

## Testing for other configurations

The example above runs for archs. To use other architectures, you can simply:
1. Change the available binaries in `$TOOLCHAIN_INST`
2. Give qemu the appropriate `$QEMU_CPU`
3. Set the correct `target_triplet` and `target_alias`

Below are some common setups.

### hs6x

1. Make available arc64-elf-* tools and necessary resources
2. export QEMU_CPU=hs6x
3. In site.exp above, set the following:

```shell
set target_triplet arc64-unknown-elf
set target_alias arc64-elf
```

### hs5x

1. Make available arc32-elf-* tools and necessary resources
2. export QEMU_CPU=hs5x
3. In site.exp above, set the following:
```shell
set target_triplet arc32-unknown-elf
set target_alias arc32-elf
```

## Using NSIM

To use nsim simply set in site.exp the following:

```shell
set target_list arc-sim-nsimdrv
```
