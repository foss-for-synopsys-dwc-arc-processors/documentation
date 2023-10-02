# IoT Development Kit

!!! info

    Please refer to board's documentation for detailed information about how to
    setup the board for initial operation:

    * [ARC IoT Development Kit](../../platforms/board-iot.md)

    Also refer [Getting OpenOCD](../../platforms/get-openocd.md) and 
    [Using OpenOCD](../../platforms/use-openocd.md) for details about installing
    and using OpenOCD.

## Building an Application

Consider a simple application with name `main.c`:

```c
#include <stdio.h>

int main()
{
    printf("Hello, World!\n");
    return 0;
}
```

Build the application:

```shell
arc-elf32-gcc -mcpu=em4_fpuda -mmpy-option=wlh5 -mfpu=fpuda_all \
              -specs=iotdk.specs main.c -o main.elf
```

`-specs=iotdk.specs` sets a proper [memory map](../general/memory.md) and links the
application with additional startup code and UART library for input/output
operations.

## Running an Application

Follow [Using OpenOCD](../../platforms/use-openocd.md) guide and start OpenOCD
with `snps_iotdk.cfg` configuration file. Here is an example:

```text
$ openocd -f board/snps_iotdk.cfg
...
```

Then connect to the server using GDB:

```text
$ arc-elf32-gdb -quiet main.elf
(gdb) target remote :3333

# Increase timeout, because OpenOCD sometimes can be slow
(gdb) set remotetimeout 15

# Load application into target
(gdb) load

# Go to start of main function
(gdb) tbreak main
(gdb) continue

# Resume with usual GDB commands
(gdb) step
(gdb) next

# Go to end of the application
(gdb) tbreak exit
(gdb) continue

# For example, check exit code of application
(gdb) info reg r0
```

## Connecting to the Serial Terminal

Follow [the corresponding guide](../../platforms/board-iot.md#connecting-to-the-serial-terminal)
for ARC IoT Development Kit.
