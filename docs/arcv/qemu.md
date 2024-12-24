# Running on QEMU

## Installing QEMU

It is best to use the latest official QEMU version for the following exercises. As of today,
it's v9.2 (see [the release notes](https://wiki.qemu.org/ChangeLog/9.2)). The sources are on
[GitLab](https://gitlab.com/qemu-project/qemu) or in [the official GitHub mirror](https://github.com/qemu/qemu).

To get the latest version of QEMU on your system it might be much easier to build it from sources
rather than trying to find a pre-built version which suits your host type and operating system.

Detailed instructions for building on Linux are on [QEMU's Wiki](https://wiki.qemu.org/Hosts/Linux).
First you need to install required additional packages, which depend on the Linux distribution used on the host.

For example, for Ubuntu 20.04 the following needs to be done:

```
$ apt install build-essential git libglib2.0-dev libfdt-dev \
              libpixman-1-dev zlib1g-dev ninja-build python3-venv
```

For AlmaLinux 8 the following needs to be done:

```
$ dnf install git glib2-devel libfdt-devel pixman-devel zlib-devel \
              bzip2 ninja-build python3 python38
```

After installation of prerequisites you can download sources, configure, and build in the following way:

```
# Clone v9.2 from GitHub repository
$ git clone --depth 1 -b stable-9.2 https://github.com/qemu/qemu.git

# Configure project for build selecting only RISCV32 full system emulation
$ ./configure --target-list=riscv32-softmmu,riscv64-softmmu

# Build QEMU
$ make
```

After the build is complete `qemu-system-riscv32` and `qemu-system-riscv64` are placed in the
`build` folder, which you can add to the `PATH` environment variable for convenience, and use
the short binary name instead of the full path.

## Code Example

Consider this code example which prints all passed command
line arguments:

```c
#include <stdio.h>

void __wrap__arcv_cache_enable() {}

int main(int argc, char *argv[]) {
    for (int i = 0; i < argc; i++) {
        printf("%d: %s\n", i, argv[i]);
    }
    return 0;
}
```

By default, ARC-V specific startup code turns on caches through
ARC-V specific CSR. However, it's not available in QEMU and to solve
this issue we added `__wrap__arcv_cache_enable` to overwrite original
caches initialization function. Also, we will have to pass an extra
`-Wl,--wrap=_arcv_cache_enable` option to GCC.

Also, code sections will be placed in `0x80000000` since in default
QEMU machine DDR is placed at `0x80000000`, where the board jumps on
start automatically.

## Build and Run for Base RMX-100

Build the application:

```
$ riscv64-snps-elf-gcc \
        -march=rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr \
        -mabi=ilp32 \
        -mtune=arc-v-rmx-100-series \
        -Wl,--wrap=_arcv_cache_enable \
        -Wl,-defsym=txtmem_addr=0x80000000 \
        -Wl,-defsym=txtmem_len=1M \
        -Wl,-defsym=datamem_addr=0x80200000 \
        -Wl,-defsym=datamem_len=1M \
        -specs=semihost.specs \
        -specs=arcv.specs \
        -T arcv.ld \
        args.c -o args.elf
```

Run on QEMU:

```
$ qemu-system-riscv32 \
        -semihosting \
        -nographic \
        -machine virt \
        -cpu rv32,f=off,d=off,zce=on,zba=on,zbb=on,zbs=on \
        -bios none \
        -kernel args.elf \
        -append "one two three"
0: args.elf
1: one
2: two
3: three
```

## Build and Run for Base RMX-500

Build the application:

```
$ riscv64-snps-elf-gcc \
        -march=rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr \
        -mabi=ilp32 \
        -mtune=arc-v-rmx-500-series \
        -Wl,--wrap=_arcv_cache_enable \
        -Wl,-defsym=txtmem_addr=0x80000000 \
        -Wl,-defsym=txtmem_len=1M \
        -Wl,-defsym=datamem_addr=0x80200000 \
        -Wl,-defsym=datamem_len=1M \
        -specs=semihost.specs \
        -specs=arcv.specs \
        -T arcv.ld \
        args.c -o args.elf
```

Run on QEMU:

```
$ qemu-system-riscv32 \
        -semihosting \
        -nographic \
        -machine virt \
        -cpu rv32,f=off,d=off,zce=on,zba=on,zbb=on,zbs=on \
        -bios none \
        -kernel args.elf \
        -append "one two three"
0: args.elf
1: one
2: two
3: three
```

## Build and Run for Base RHX-100

Build the application:

```
$ riscv64-snps-elf-gcc \
        -march=rv32imac_zcb_zcmp_zba_zbb_zbs_zicsr \
        -mabi=ilp32 \
        -mtune=arc-v-rhx-100-series \
        -Wl,--wrap=_arcv_cache_enable \
        -Wl,-defsym=txtmem_addr=0x80000000 \
        -Wl,-defsym=txtmem_len=1M \
        -Wl,-defsym=datamem_addr=0x80200000 \
        -Wl,-defsym=datamem_len=1M \
        -specs=semihost.specs \
        -specs=arcv.specs \
        -T arcv.ld \
        args.c -o args.elf
```

Run on QEMU:

```
$ qemu-system-riscv32 \
        -semihosting \
        -nographic \
        -machine virt \
        -cpu rv32,f=off,d=off,m=on,a=on,zca=on,zcb=on,zcmp=on,zba=on,zbb=on,zbs=on \
        -bios none \
        -kernel args.elf \
        -append "one two three"
0: args.elf
1: one
2: two
3: three
```

## Build and Run for Base RPX-100

Build the application:

```
$ riscv64-snps-elf-gcc \
        -march=rv64imac_zcb_zba_zbb_zbs_zicsr \
        -mabi=lp64 \
        -mtune=arc-v-rmx-100-series \
        -mcmodel=medany \
        -Wl,--wrap=_arcv_cache_enable \
        -Wl,-defsym=txtmem_addr=0x80000000 \
        -Wl,-defsym=txtmem_len=1M \
        -Wl,-defsym=datamem_addr=0x80200000 \
        -Wl,-defsym=datamem_len=1M \
        -specs=semihost.specs \
        -specs=arcv.specs \
        -T arcv.ld \
        args.c -o args.elf
```

Run on QEMU:

```
$ qemu-system-riscv64 \
        -semihosting \
        -nographic \
        -machine virt \
        -cpu rv64,f=off,d=off,m=on,a=on,zca=on,zcb=on,zba=on,zbb=on,zbs=on \
        -bios none \
        -kernel args.elf \
        -append "one two three"
0: args.elf
1: one
2: two
3: three
```

## Debugging Using GDB

You can debug an application that is being run on QEMU with help of the built-in GDB server of
QEMU. On the QEMU side, you need to enable the GDB sever and optionally instruct the QEMU not 
to start the application execution automatically, but instead wait for user input. For that you
need to pass `-s -S` commands to `qemu-system-riscv32` or `qemu-system-riscv64`.

Build the same example for RMX-100:

```
$ riscv64-snps-elf-gcc \
        -march=rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr \
        -mabi=ilp32 \
        -mtune=arc-v-rmx-100-series \
        -Wl,--wrap=_arcv_cache_enable \
        -Wl,-defsym=txtmem_addr=0x80000000 \
        -Wl,-defsym=txtmem_len=1M \
        -Wl,-defsym=datamem_addr=0x80200000 \
        -Wl,-defsym=datamem_len=1M \
        -specs=semihost.specs \
        -specs=arcv.specs \
        -T arcv.ld \
        -g \
        args.c -o args.elf
```

Start GDB server (we use `-gdb tcp::12345` instead of `-s` to set the custom
GDB port number instead of the default 1234 port number):

```
$ qemu-system-riscv32 \
        -semihosting \
        -nographic \
        -machine virt \
        -cpu rv32,f=off,d=off,zce=on,zba=on,zbb=on,zbs=on \
        -bios none \
        -kernel args.elf \
        -append "one two three" \
        -gdb tcp::12345 -S
```

Start GDB session:

```
$ riscv64-snps-elf-gdb -q args.elf
Reading symbols from args.elf...
(gdb) target remote :12345
Remote debugging using :12345
0x00001000 in ?? ()
(gdb) b main
Breakpoint 1 at 0x800000d6: file args.c, line 8.
(gdb) c
Continuing.

Breakpoint 1, main (argc=4, argv=0x80200e78 <_argv>) at args.c:8
8           for (int i = 0; i < argc; i++) {
(gdb) step
9               printf("%d: %s\n", i, argv[i]);
(gdb) c
Continuing.
[Inferior 1 (process 1) exited normally]
```

Note that the first location where the CPU is waiting for commands is `0x00001000`. It’s a
`virt` board reset vector location. It is not possible to suppress execution of that reset
vector code as it’s an essential part of the QEMU board (think of it as a boot-ROM). You
can only bypass it if you load the target executable from the GDB client with the load command.
Then execution starts from the loaded application entry point.
