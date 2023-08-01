# Running Newlib Tests

## Preface

After building the toolchain using the
[arc-gnu-toolchain](https://github.com/foss-for-synopsys-dwc-arc-processors/arc-gnu-toolchain)
repository, all tests are automatically available as per
[this section in the readme](https://github.com/foss-for-synopsys-dwc-arc-processors/arc-gnu-toolchain#testing-gccc-binutils-and-newlib).

## Standalone testing

It might be useful to perform tests without having to build the entire toolchain.

The newlib tests are based on [dejagnu](https://www.gnu.org/software/dejagnu/) so it is
expected that the framework is installed in your system.

Depending on the simulator used (nSIM or [QEMU](https://github.com/foss-for-synopsys-dwc-arc-processors/qemu))
it is expected that the correct binaries are present in the `PATH`.

The most direct (manual) way to run the tests is to setup the local environment,
include the necessary sources and binaries, and run using a site.exp file. A simple
test procedure goes as follows:

```shell
$ mkdir test_env; cd test_env           # There are output binaries and logs, so its' useful to keep them separate
$ mkdir tmp                             # Directory used to store the generated objects
$ export TOOLCHAIN_SRC=/path/to/parent  # Path to parent folder of all ARC tool sources (at least newlib, toolchain and arc-gnu-toolchain)
$ export TOOLCHAIN_INST=/path/to/bins   # Path to installed binaries (only required if not in path already)
$ export QEMU_CPU=archs                 # CPU to use in simulation
$ export PATH=$PATH:$TOOLCHAIN_INST/bin:$TOOLCHAIN_SRC/arc-gnu-toolchain/scripts/wrapper/qemu # Add tools and QEMU wrapper to path (used by arc-sim board)
$ cat > site.exp << 'EOF'
set tool newlib
set tool_version 4.3.0

set tmpdir "./tmp"
set srcdir "$env(TOOLCHAIN_SRC)/newlib/newlib/testsuite"

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

## Testing for Other Configurations

The example above runs for archs. To use other architectures, you can simply:

1. Change the available binaries in `$TOOLCHAIN_INST`
2. Give qemu the appropriate `$QEMU_CPU`
3. Set the correct `target_triplet` and `target_alias`

Below are some common setups.

## Running Tests for HS6x

1. Make available arc64-elf-* tools and necessary resources
2. export QEMU_CPU=hs6x
3. In site.exp above, set the following:

```shell
set target_triplet arc64-unknown-elf
set target_alias arc64-elf
```

## Running Tests for HS5x

1. Make available arc32-elf-* tools and necessary resources
2. export QEMU_CPU=hs5x
3. In site.exp above, set the following:

```shell
set target_triplet arc32-unknown-elf
set target_alias arc32-elf
```

## Running Tests using nSIM

To use nsim simply set in site.exp the following:

```shell
set target_list arc-sim-nsimdrv
```
