# QEMU

## Running Applications

* [Building Baremetal Applications and Debugging on QEMU](../baremetal/simulators/qemu.md)
* [Building Linux and Running on QEMU](../linux/simulators/qemu.md)

## Building QEMU

!!! info "Additional packages and plugins"

    Refer to [the official documentation](https://wiki.qemu.org/Hosts/Linux#Required_additional_packages) for details about
    installing additional packages and enabling support of QEMU plugins.

Install necessary packages for Arch:

```shell
sudo pacman -S git cmake ninja gperf ccache dfu-util dtc wget            \
               python-pip python-setuptools python-wheel xz file make
```

Install necessary packages for Fedora:

```shell
sudo dnf group install "Development Tools" "C Development Tools and Libraries"
dnf install git cmake ninja-build gperf ccache dfu-util dtc wget         \
            python3-pip xz file glibc-devel.i686 libstdc++-devel.i686
```

Install necessary packages for Ubuntu:

```shell
sudo apt-get install --no-install-recommends git cmake ninja-build gperf \
                     ccache dfu-util device-tree-compiler wget           \
                     python3-pip python3-setuptools python3-wheel        \
                     xz-utils file make gcc gcc-multilib
```

Install necessary packages for Void:

```shell
sudo xbps-install git cmake ninja gperf ccache dfu-util dtc wget  \
                  python3-pip python3-setuptools python3-wheel xz \
                  file make
```

Then prepare sources and a build directory:

```bash
git clone https://github.com/foss-for-synopsys-dwc-arc-processors/qemu
mkdir -p qemu/build
cd qemu/build
```

Configure QEMU inside of the build directory (use your own `--prefix` value for installation path):

```
../configure --target-list=arc-softmmu,arc64-softmmu,arc-linux-user,arc64-linux-user \
             --prefix=/tools/qemu --enable-debug --enable-debug-tcg --enable-trace-backends=simple \
             --disable-plugins --skip-meson --disable-pie
```

If you face an error then consider using `--disable-werror` configure option to
try to eliminate it. Also, consider reporting a bug on
[toolchain's GitHub page](https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain/issues/new).

What options are responsible for what:

* `--target-list=arc-softmmu,arc64-softmmu,arc-linux-user,arc64-linux-user` — build QEMU both for these targets:
  * `qemu-system-arc` — system emulation for ARC HS3x/4x/5x processors family;
  * `qemu-system-arc64` — system emulation for ARC HS6x processors family;
  * `qemu-arc` — user space Linux emulation for ARC HS3x/4x/5x processors family;
  * `qemu-arc64` — user space Linux emulation for ARC HS6x processors family.
* `--prefix=/tools/qemu` — an installation path.
* `--enable-debug --enable-debug-tcg --enable-trace-backends=simple --disable-plugins` — options for development needs.
* `--enable-trace-backends=simple` — for tracing (described in [[Profiling with QEMU]]).
* `--skip-meson` — do not run Meson on every build.
* `--disable-werror` — in case QEMU emits unexpected warnings.
* `--disable-pie` — needed for older GCC (like in CentOS 7).

Build and install:

```shell
make
make install
```

Configure your environment:

```bash
export QEMU_HOME="/tools/qemu"
export PATH="${QEMU_HOME}/bin:$PATH"
```

## Enhanced Logging

To enable logging, it is necessary to provide the enabled log levels with the `-d` flag. Some of the more relevant ones are:

```text
in_asm          show target assembly code for each compiled TB
nochain         do not chain compiled TBs so that "exec" and "cpu" show complete traces
exec            show trace before each executed TB (lots of logs)
cpu             show CPU registers before entering a TB (lots of logs)
fpu             include FPU registers in the 'cpu' logging
int             show interrupts/exceptions in short format
mmu             log MMU-related activities
unimp           log unimplemented functionality
```

To get a complete listing, run `qemu-system-arc -d help`. Use `-D <logfile>` to dump the logs into a file instead of
standard output.

## Tracing Internals

QEMU provides a tracing infrastructure which may help in debugging or analyzing what happens within a simulation cycle.
At this moment, there are two tracers added into ARC backend, one for MMU operations, and another for exceptions:

```c
# mmu.c
mmu_command(uint32_t address, const char *command, uint32_t pd0, uint32_t pd1) "[MMU] at 0x%08x, CMD=%s, PD0=0x%08x, PD1=0x%08x"

# helper.c
excp_info(uint32_t address, const char *name) "[IRQ] at 0x08, Exception=%s"
```

Firstly, build QEMU with the `--enable-trace-backends=simple` configure parameter. Then
Create a file with the events you want to trace. For example, here is such file with name `events.trc`:

```text
mmu_command
excp_info
```

Run the virtual machine to produce a trace file:

```shell
qemu-system-arc --trace events=events.trc ...
```

Pretty-print the binary trace file (override `<pid>` with QEMU process id for you session):

```text
<QEMU-source-tree-path>/scripts/simpletrace.py <QEMU-source-tree-path>/target/arc/trace-events trace-<pid>
```

## Running Tests

TCG is the internal language that powers QEMU. There are some assembly tests
that validate the basic function of several instructions in QEMU.

Firstly, make sure that QEMU if configured with `--cross-cc-arc=arc-elf32-gcc`
and `--cross-cc-arc64=arc64-elf-gcc` options. Then after building QEMU use these
commands to run TCG tests:

```shell
make clean-tcg
make build-tcg
make check-tcg
```
