# Cross-compiling eBPF Programs

!!! info

    You can use [the testbench](https://github.com/foss-for-synopsys-dwc-arc-processors/arc-bpf-testbench)
    that automates all the above steps for building eBPF programs for ARC.

## Preface

There are several ways of building eBPF applications for Linux. The main and
most straightforward way consists of these steps:

1. Build the eBPF part using Clang (it's a regular Object in ELF format).
2. Generate a special C file named "skeleton" which contains an entire blob
   for the eBPF part and the functions for loading it using `libbpf`.
3. Write a C program which uses these functions for opening the eBPF
   application, loading it into the kernel and attaching it to events.
4. Build your C program using GCC toolchain and run on your target.

## Building eBPF Programs Natively (e.g. x86 host and target)

Consider the `minimal` program from `libbpf-bootstrap`
[repository](https://github.com/libbpf/libbpf-bootstrap). It consists of
[2 file](https://github.com/libbpf/libbpf-bootstrap/tree/master/examples/c):
`minimal.c` and `minimal.bpf.c`. Ensure that `libbpf` and the
corresponding development files are installed (e.g., for Fedora you need to
install `libbpf` and `libbpf-devel` packages). Also, a recent version of
`bpftool` utility must be installed. Then you can build the program using
these commands (replace `-D__TARGET_ARCH_x86` by an appropriate one for your
platform):

```shell
clang -g                  \
      -O2                 \
      -Wall               \
      -target bpf         \
      -D__TARGET_ARCH_x86 \
      -c minimal.bpf.c    \
      -o minimal.bpf.o

bpftool gen skeleton minimal.bpf.o > minimal.skel.h

gcc minimal.c -lbpf       \
              -lelf       \
              -lz         \
              -o minimal
```

Then you can run this program with `root` privileges:

```text
# The 2 commands below are a one-time setup. Run them with root privileges.
mount -t debugfs debugfs /sys/kernel/debug
sysctl net.core.bpf_jit_enable=1           # Keep 0, if you don't want JIT.

$ sudo ./minimal 
[sudo] password for user: 
libbpf: loading object 'minimal_bpf' from buffer
libbpf: elf: section(3) tp/syscalls/sys_enter_write, size 104, link 0, flags 6, type=1
libbpf: sec 'tp/syscalls/sys_enter_write': found program 'handle_tp' at insn offset 0 (0 bytes), code size 13 insns (104 bytes)
libbpf: elf: section(4) .reltp/syscalls/sys_enter_write, size 32, link 22, flags 40, type=9
libbpf: elf: section(5) license, size 13, link 0, flags 3, type=1
libbpf: license of minimal_bpf is Dual BSD/GPL
libbpf: elf: section(6) .bss, size 4, link 0, flags 3, type=8
libbpf: elf: section(7) .rodata, size 28, link 0, flags 2, type=1
libbpf: elf: section(13) .BTF, size 609, link 0, flags 0, type=1
libbpf: elf: section(15) .BTF.ext, size 160, link 0, flags 0, type=1
libbpf: elf: section(22) .symtab, size 336, link 1, flags 0, type=2
libbpf: looking for externs among 14 symbols...
libbpf: collected 0 externs total
libbpf: map 'minimal_.bss' (global data): at sec_idx 6, offset 0, flags 400.
libbpf: map 0 is "minimal_.bss"
libbpf: map 'minimal_.rodata' (global data): at sec_idx 7, offset 0, flags 480.
libbpf: map 1 is "minimal_.rodata"
libbpf: sec '.reltp/syscalls/sys_enter_write': collecting relocation for section(3) 'tp/syscalls/sys_enter_write'
libbpf: sec '.reltp/syscalls/sys_enter_write': relo #0: insn #2 against 'my_pid'
libbpf: prog 'handle_tp': found data map 0 (minimal_.bss, sec 6, off 0) for insn 2
libbpf: sec '.reltp/syscalls/sys_enter_write': relo #1: insn #6 against '.rodata'
libbpf: prog 'handle_tp': found data map 1 (minimal_.rodata, sec 7, off 0) for insn 6
libbpf: map 'minimal_.bss': created successfully, fd=4
libbpf: map 'minimal_.rodata': created successfully, fd=5
Successfully started! Please run `sudo cat /sys/kernel/debug/tracing/trace_pipe` to see output of the BPF programs.
```

Some examples from `libbpf-bootstrap` use `vmlinux.h` - it's a header file
with the definitions of all the data structures for that particular kernel. The
easiest way to obtain it on the host system is by using `bpftool`, if the
kernel was built with `CONFIG_DEBUG_INFO_BTF=y`:

```shell
bpftool --debug btf dump file /sys/kernel/btf/vmlinux format c > vmlinux.h
```

Else, you can generate `vmlinux.h` from any `vmlinux` image with BTF
information:

```shell
bpftool --debug btf dump file path/to/your/vmlinux format c > vmlinux.h
```

## Building eBPF Programs and Their Dependencies for ARC

Suppose that the root directory of the toolchain for ARC HS 4x is
`/tools/arc-linux-gnu`. `/tools/arc-linux-gnu/bin` must be in `PATH`
variable. `bpftool` must be available too.

Create a directory for ARC's libraries and includes:

```shell
mkdir /tools/sysroot
```

Export common variables:

```shell
export CFLAGS="-Og -g3 -fPIC"
export CXXFLAGS="-Og -g3 -fPIC"
export CPPFLAGS="-I/tools/sysroot/include"
export LDFLAGS="-L/tools/sysroot/lib"
```

We are going to use the latest releases of libraries at the moment of writing
of this manual instead of development branches.

### Building zlib

```shell
git clone -b v1.2.13 https://github.com/madler/zlib
cd zlib
./configure CHOST="arc-linux-gnu" --static --prefix="/tools/sysroot"
make install
```

### Building libelf

```shell
git clone -b elfutils-0.189 https://sourceware.org/git/elfutils.git
cd elfutils
autoreconf -i -f

./configure --host=arc-linux-gnu          \
            --target=arc-linux-gnu        \
            --disable-dependency-tracking \
            --disable-nls                 \
            --program-prefix="eu-"        \
            --disable-libdebuginfod       \
            --disable-debuginfod          \
            --without-bzlib               \
            --without-lzma                \
            --without-zstd                \
            --prefix="/tools/sysroot"     \
            --enable-maintainer-mode

make -C libelf install-includeHEADERS install-libLIBRARIES
```

### Building libbpf

```shell
git clone -b v1.1.0 https://github.com/libbpf/libbpf
cd libbpf/src

make BUILD_STATIC_ONLY="y"          \
     PREFIX="/"                     \
     DESTDIR="/tools/sysroot"       \
     CROSS_COMPILE="arc-linux-gnu-" \
     install install_uapi_headers
```

### Generating ``vmlinux.h`` from your ``vmlinux`` image

Make sure that you have a recent version of bpftool (≥7).

```shell
bpftool --debug btf dump file path/to/your/vmlinux format c > /tools/sysroot/include/vmlinux.h
```

### Overview of `/tools/sysroot`

After building and installing all libraries `/tools/sysroot` should look like this:

```text
$ tree -L 3 /tools/sysroot
/tools/sysroot
|-- include
|   |-- bpf
|   |   |-- bpf_core_read.h
|   |   |-- bpf_endian.h
|   |   |-- bpf.h
|   |   |-- bpf_helper_defs.h
|   |   |-- bpf_helpers.h
|   |   |-- bpf_tracing.h
|   |   |-- btf.h
|   |   |-- libbpf_common.h
|   |   |-- libbpf.h
|   |   |-- libbpf_legacy.h
|   |   |-- libbpf_version.h
|   |   |-- skel_internal.h
|   |   `-- usdt.bpf.h
|   |-- gelf.h
|   |-- libelf.h
|   |-- linux
|   |   |-- bpf.h
|   |   |-- bpf_common.h
|   |   `-- btf.h
|   |-- nlist.h
|   |-- zconf.h
|   `-- zlib.h
|-- lib
|   |-- libbpf.a
|   |-- libelf.a
|   |-- libz.a
|   `-- pkgconfig
|       |-- libbpf.pc
|       `-- zlib.pc
`-- share
    `-- man
        `-- man3

9 directories, 26 files
```

### Building eBPF Program

```shell
# The paths where the native ARC tools and the cross ARC toolchain are installed
readonly NATIVE_SYSROOT=/tools/sysroot
readonly TOOLCHAIN_SYSROOT=/tools/arc-linux-gnu/sysroot

# Clang uses host's Linux headers if we don't pass toolchain's includes.
# Also, the -XClang arguments below are optional.
clang -g                                 \
      -O2                                \
      -Wall                              \
      -target bpf                        \
      -D__TARGET_ARCH_arc                \
      -I${NATIVE_SYSROOT}/usr/include    \
      -I${TOOLCHAIN_SYSROOT}/usr/include \
      -Xclang -target-feature            \
      -Xclang +alu32                     \
      -c minimal.bpf.c                   \
      -o minimal.bpf.o

# bpftool must be a recent version (≥7)
bpftool gen skeleton minimal.bpf.o > minimal.skel.h

# shared library build (-latomic is listed for future reference)
arc-linux-gcc minimal.c                       \
              -I${NATIVE_SYSROOT}/usr/include \
              -L${NATIVE_SYSROOT}/usr/lib     \
              -lbpf                           \
              -lelf                           \
              -lz                             \
              -latomic                        \
              -o minimal

# static build (-latomic is listed for future reference)
arc-linux-gcc minimal.c                          \
              -I${SYSROOT}/usr/include           \
              ${NATIVE_SYSROOT}/usr/lib/libbpf.a \
              ${NATIVE_SYSROOT}/usr/lib/libelf.a \
              ${NATIVE_SYSROOT}/usr/lib/libz.a   \
              -latomic                           \
              -o minimal
```
