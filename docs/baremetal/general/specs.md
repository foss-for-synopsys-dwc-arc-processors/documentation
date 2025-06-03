# Specs Files

## Overview

There is a set of files called _specs files_ with `.specs` extension. They are
shipped with Newlib library in the baremetal toolchain. A specs file for a
particular platform (for example, a simulator or a development board) is
responsible for linking the application with specific startup code and
additional libraries.

For example, a specs file for HS Development Kit is called `hsdk.specs` and
it does this when it's used:

1. Link an application with a custom linker script that represents HSDK's
   [memory map](./memory.md).
2. Link an application with a custom startup code.
3. Link an application with UART library for input/output operations.

You can use specs file using `-specs=` option:

```shell
arc-elf32-gcc -mcpu=archs -specs=hsdk.specs main.c -o main.elf
```

## List of Specs Files

There is a list of general specs files:

* `nosys.specs` - link stubs for system calls.
* `nsim.specs` - link with GNU hostlink library for nSIM simulator (with `-on nsim_emt`) or QEMU (`-semihosting`).
* `hl.specs` - link with MetaWare hostlink library for nSIM simulator (default hostlink mode).
* `nano.specs` - use Newlib nano instead of default Newlib implementation. May be used
  in conjunction with other specs file. For example: `-specs=nano.specs -specs=nsim.specs`. 

Also, there are specs files for various development platforms:

* [HS Development Kit](../hardware/hsdk.md)
* [EM Software Development Platform](../hardware/emsdp.md)
