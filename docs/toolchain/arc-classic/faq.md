# Frequently Asked Questions

## I’ve opened `hs38.tcf` and GCC options include `-mcpu=hs34`. Why `hs34` instead of `hs38`?

Possible values of `-mcpu=` options are orthogonal to names of IPlib templates and respective TCF.
GCC option `-mcpu=` supports both `hs34` and `hs38` values, but they are different - `hs38` enables
more features, like `-mll64` which are not present in `hs34`. ARC HS IPlib template `hs38` doesn’t
contain double-word load/store, therefore -mcpu=hs38 is not compatible with this template.
`-mcpu=hs34`, however, is compatible and that is why TCF generator uses this value. Refer
[Target Options](../index.md) sections for a full list of possible `-mcpu`
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
