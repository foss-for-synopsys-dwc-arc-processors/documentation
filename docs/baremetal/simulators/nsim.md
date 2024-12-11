# Building Applications and Debugging on nSIM

nSIM supports running and debugging applications built with GNU toolchain for all ARC families.

There is no a definite match between TCF templates for nSIM and `-mcpu` values for GNU toolchain.
If you want to build a program using GNU toolchain and run it on nSIM simulator , then you have two
ways of finding right nSIM options:

1. Find proper nSIM options in corresponding [Target Options](../../toolchain/index.md) sections for
   different targets.
2. Use default TCF templates and additional nSIM options. You can find matching templates in the same
   sections of [Target Options](../../toolchain/index.md).

Note that you need to pass `-on nsim_isa_big_endian` to nSIM for big endian targets.

## Building and Debugging Applications

Suppose that `main.c` contains an application to be debugged on nSIM:

```c
int main()
{
    return 0;
}
```

Then build it (we use `-specs=nosys.specs` if input/output operations are not needed) for ARC HS3x using `-mcpu=hs38`:

```
$ arc-snps-elf-gcc -mcpu=hs38 -specs=nosys.specs -g main.c -o main.elf
```

Start nSIM with a GDB server with 12345 port:

```
$ nsimdrv -tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf -gdb -port 12345
```

Run GDB in another terminal and debug the application:

```
$ arc-snps-elf-gdb -quiet main.elf
Reading symbols from main.elf...
(gdb) target remote :12345
Remote debugging using :12345
0x80000000 in ?? ()
(gdb) load
Loading section .init, size 0x22 lma 0x100
Loading section .text, size 0x1548 lma 0x124
Loading section .fini, size 0x16 lma 0x166c
Loading section .ivt, size 0x54 lma 0x1682
Loading section .data, size 0x534 lma 0x36d8
Loading section .ctors, size 0x8 lma 0x3c0c
Loading section .dtors, size 0x8 lma 0x3c14
Loading section .sdata, size 0x10 lma 0x3c1c
Start address 0x00000124, load size 6952
Transfer rate: 188 KB/sec, 869 bytes/write.
(gdb) b main
Breakpoint 1 at 0x276: file main.c, line 3.
(gdb) c
Continuing.

Breakpoint 1, main () at main.c:3
3           return 0;
(gdb)
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

You need to use `-specs=nsim.specs` to use input/output features and to pass `-on nsim_emt` option to nSIM
to use ARC GNU input/output protocol:

```
$ arc-snps-elf-gcc -mcpu=archs -specs=nsim.specs main.c -o main.elf
$ nsimdrv -tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf -on nsim_emt main.elf
Hello, World!
```

You can use MetaWare's own hostlink protocol for input/output operations by passing `-specs=hl.specs` to GCC.
In this case you don't have to pass any additional options to nSIM:

```
$ arc-snps-elf-gcc -mcpu=archs -specs=hl.specs main.c -o main.elf
$ nsimdrv -tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf main.elf
Hello, World!
```
