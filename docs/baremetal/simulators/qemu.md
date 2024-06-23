# Building Applications and Debugging on QEMU

!!! info

    Refer to the main [QEMU](../../simulators/qemu.md) article for information
    on installing QEMU.

## Building and Debugging Applications

QEMU supports running and debugging applications for ARC HS3x/HS4x, HS5x and HS6x families. Here is a
table of tools and options for a particular family:

| CPU family    | Toolchain       | `-mcpu=`      | QEMU binary         | ARC-specific QEMU options |
|---------------|-----------------|---------------|---------------------|---------------------------|
| ARC HS3x/HS4x | `arc-elf32-gcc` | `-mcpu=archs` | `qemu-system-arc`   | `-M arc-sim -cpu archs`   |
| ARC HS5x      | `arc64-elf-gcc` | `-mcpu=hs5x`  | `qemu-system-arc`   | `-M arc-sim -cpu hs5x`    |
| ARC HS6x      | `arc64-elf-gcc` | `-mcpu=hs6x`  | `qemu-system-arc64` | `-M arc-sim -cpu hs6x`    |

Suppose that `main.c` contains an application to be debugged on QEMU for ARC HS3x/4x:

```c
int main()
{
    return 0;
}
```

Then build it (we use `-specs=nosys.specs` if input/output operations are not needed):

```shell
arc-elf32-gcc -mcpu=archs -specs=nosys.specs -g main.c -o main.elf
```

Start a GDB server in port 1234 (this is the default port, so we could use the alias `-s` instead of `-gdb tcp::1234`):

```shell
qemu-system-arc -M arc-sim \
                -cpu archs \
                -monitor none \
                -display none \
                -nographic \
                -no-reboot \
                -gdb tcp::1234 -S \
                -kernel main.elf
```

Debug the application:

```text
$ arc-elf32-gdb -quiet main.elf
Reading symbols from main.elf...
(gdb) target remote :1234
Remote debugging using :1234
0x00000124 in __start ()
(gdb) load
Loading section .init, size 0x22 lma 0x100
Loading section .text, size 0x1554 lma 0x124
Loading section .fini, size 0x16 lma 0x1678
Loading section .ivt, size 0x54 lma 0x168e
Loading section .data, size 0x534 lma 0x36e8
Loading section .ctors, size 0x8 lma 0x3c1c
Loading section .dtors, size 0x8 lma 0x3c24
Loading section .sdata, size 0x10 lma 0x3c2c
Start address 0x00000124, load size 6964
Transfer rate: 1700 KB/sec, 696 bytes/write.
(gdb) b main
Breakpoint 1 at 0x276: file main.c, line 3.
(gdb) c
Continuing.

Breakpoint 1, main () at main.c:3
3               return 0;
(gdb)
```

## Using a Socket Instead of Port

If known ports are busy then you can connect to the GDB server using a socket.
Expose GDB server through socket instead of port

```shell
qemu-system-arc -M arc-sim \
                -cpu archs \
                -monitor none \
                -display none \
                -nographic \
                -no-reboot \
                -chardev socket,path=/tmp/gdb-socket,server=on,wait=off,id=gdb0 \
                -gdb chardev:gdb0 -S \
                -kernel main.elf
```

Connect to the GDB server
```text
$ arc-elf32-gdb -quiet main.elf
Reading symbols from main.elf...
(gdb) target remote /tmp/gdb-socket
Remote debugging using /tmp/gdb-socket
0x00000124 in __start ()
...
```

## Building and Running "Hello, World!"

Consider a simple example code (save it as `main.c`):

```c
#include <stdio.h>

int main()
{
    printf("Hello, World!\n");
    return 0;
}
```

You need to use `-specs=nsim.specs` to use input/output features and
pass `-semihosting` option to QEMU:

```text
$ arc-elf32-gcc -mcpu=archs -specs=nsim.specs main.c -o main.elf
$ qemu-system-arc -M arc-sim \
                  -cpu archs \
                  -monitor none \
                  -display none \
                  -nographic \
                  -no-reboot \
                  -semihosting \
                  -kernel main.elf
Hello, World!
```

## Building "Hello, World!" Using MetaWare Development Toolkit and Running on QEMU

Without `-semihosting` QEMU creates a character device for `arc-sim` board on
hard coded `0x90000000` address. MetaWare’s standard runtime library does not
support input/output interfaces of QEMU for ARC. But it is possible to implement
your own basic `hostlink` library for MetaWare to meet QEMU's requirements.
You have to implement at least one function to add support of simple output
using the character device at `0x90000000`:

```c
int _write (int handle, const char *buf, unsigned int count)
{
    unsigned int i = 0;
    while (i < count)
    {
        *(char *) 0x90000000 = buf[i++];
    }
    return count;
}
```

Save it as `write.c` file and compile it along with `main.c`:

```shell
# For ARC HS3x/HS4x
ccac -av2hs -Hhostlib= main.c write.c -o main.elf

# For ARC HS5x
ccac -av3hs -Hhostlib= main.c write.c -o main.elf

# For ARC HS6x
ccac -arc64 -Hhostlib= main.c write.c -o main.elf
```

Run it using QEMU with `-serial stdio` option:

```shell
# For ARC HS3x/HS4x
qemu-system-arc -M arc-sim \
                -cpu archs \
                -monitor none \
                -display none \
                -nographic \
                -no-reboot \
                -serial stdio \
                -kernel main.elf

# For ARC HS5x
qemu-system-arc -M arc-sim \
                -cpu hs5x \
                -monitor none \
                -display none \
                -nographic \
                -no-reboot \
                -serial stdio \
                -kernel main.elf

# For ARC HS6x
qemu-system-arc64 -M arc-sim \
                  -cpu hs6x \
                  -monitor none \
                  -display none \
                  -nographic \
                  -no-reboot \
                  -serial stdio \
                  -kernel main.elf
```
