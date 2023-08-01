# Frequently Asked Questions

## Linker fails with error: ``undefined reference to _exit``

Among other possible functions are also `_sbrk`, `_write`, `_close`, `_lseek`,
`_read`, `_fstat`, `_isatty`.

Function `_exit` is not provided by the `libc` itself, but must be provided by the `libgloss`,
which is basically a BSP (board support package). Currently several `libgloss` implementations
are provided for ARC:

* `libnosys` - a generic implementation with stubs.
* `libnsim` - implements nSIM and QEMU IO hostlink (GNU version).
* `libqemu` - implements a simplified IO hostlink for QEMU.
* `libhl` - implements nSIM IO hostlink (MetaWare version).
* `libiotdk_uart`, `libhsdk_uart`, `libemsk_uart` - implementations for development boards.

In general `libnosys` is more suitable for hardware targets that does not have hostlink support, however
other implementations have a distinct advantage that on exit from an application and in case of many errors
it halts the core, while `libnosys` causes it to infinite loop on one place.

## I’ve opened `hs38.tcf` and GCC options include `-mcpu=hs34`. Why `hs34` instead of `hs38`?

Possible values of `-mcpu=` options are orthogonal to names of IPlib templates and respective TCF.
GCC option `-mcpu=` supports both `hs34` and `hs38` values, but they are different - `hs38` enables
more features, like `-mll64` which are not present in `hs34`. ARC HS IPlib template `hs38` doesn’t
contain double-word load/store, therefore -mcpu=hs38 is not compatible with this template.
`-mcpu=hs34`, however, is compatible and that is why TCF generator uses this value. Refer
[Target Specific Options](general/target-options.md) page for a full list of possible `-mcpu`
values and what IPlibrary templates they correspond to.

## There are `can’t resolve symbol` error messages when using `gdbserver` on Linux for ARC targets

This error message might appear when `gdbserver` is a statically linked application. Even though it
is linked statically, `gdbserver` still opens `libthread_db.so` library using `dlopen()` function.
There is a circular dependency here, as `libthread_db.so` expects several dynamic symbols to be already defined in the loading application (`gdbserver` in this case). However statically linked `gdbserver` does not export those dynamic symbols,
therefore `dlopen()` invocation causes those error messages. In practice there have not been noticed any downside of
this, even when debugging applications with threads, however that was tried only with simple test cases. To fix
this issue, either rebuild gdbserver as a dynamically linked application, or pass option
`--with-libthread-db=-lthread_db` to configure script of script. In this case gdbserver will link
with `libthread_db` statically, instead of opening it with `dlopen()` and dependency on symbols will
be resolved at link time.

## GDB prints an error message that `XML support has been disabled at compile time`

GDB uses Expat library to parse XML files. Support of XML files is optional for GDB, therefore it can
be built without Expat available, however for ARC it usually required to have support of XML to read
target description files. Mentioned error message might happen if GDB has been built without available
development files for the Expat. On Linux systems those should be available as package in package manager.

## How to reset ARC SDP board programmatically (without pressing `Reset` button)?

It is possible to reset ARC SDP board without touching the physical button on the board.
This can be done using the special OpenOCD script:

```shell
$ openocd -f test/arc/reset_sdp.tcl
```

Note that OpenOCD will crash with a segmentation fault after executing this script - this is expected
and happens only after board has been reset, but that means that other OpenOCD scripts cannot be used
in chain with `reset_sdp.tcl`, first OpenOCD should be invoked to reset the board, second it should be
invoked to run as an actual debugger.

## Can I program FPGA's in ARC EM Starter Kit or in ARC SDP?

OpenOCD has some support for programming of FPGA’s over JTAG, however it is not officially
supported for ARC development systems.

## When debugging ARC EM core in AXS101 with Ashling Opella-XD and GDBserver I get an error messages and GDB shows that all memory and registers are zeroes

Decrease a JTAG frequency to no more than 5MHz using an Ashling GDBserver option `--jtag-frequency`.
This particular problem can be noted if GDBserver prints:

```text
Error: Core is running (unexpected), attempting to halt...
Error: Core is running (unexpected), attempting to halt...
Error: Unable to halt core
```

While GDB shows that whole memory is just zeroes and all register values are also zeroes.
