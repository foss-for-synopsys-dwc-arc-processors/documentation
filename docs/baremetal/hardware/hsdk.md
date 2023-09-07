# HS Development Kit

!!! info

    Please refer to board's documentation for detailed information about how to
    setup the board for initial operation:

    * [ARC HS Development Kit 4xD](../../platforms/board-hsdk-4xd.md)
    * [ARC HS Development Kit](../../platforms/board-hsdk.md)

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
arc-elf32-gcc -mcpu=hs38_linux -specs=hsdk.specs main.c -o main.elf
```

`-specs=hsdk.specs` sets a proper [memory map](../general/memory.md) and links the
application with additional startup code and UART library for input/output
operations.

## Running an Application

!!! warning

    If `snps_hsdk_4xd.cfg` configuration file us not available in your
    installation, then consider using `snps_hsdk.cfg`.

Follow [Using OpenOCD](../../platforms/use-openocd.md) guide and start OpenOCD
with 49101 port and `snps_hsdk.cfg` (for HSDK) of `snps_hsdk_4xd.cfg`
(for HSDK 4xD) configuration file. Here is a possible output for HSDK 4xD:

```text
$ openocd -f board/snps_hsdk_4xd.cfg
Open On-Chip Debugger 0.12.0+dev-gffa52f0e0 (2023-08-02-10:41)
Licensed under GNU GPL v2
For bug reports, read
    http://openocd.org/doc/doxygen/bugs.html
Info : target has l2 cache enabled is enabled
Info : target has l2 cache enabled is enabled
Info : target has l2 cache enabled is enabled
Info : target has l2 cache enabled is enabled
32768
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : ftdi: if you experience problems at higher adapter clocks, try the command "ftdi tdo_sample_edge falling"
Info : clock speed 10000 kHz
Info : JTAG tap: arc-em.cpu4 tap/device found: 0x100c54b1 (mfg: 0x258 (ARC International), part: 0x00c5, ver: 0x1)
Info : JTAG tap: arc-em.cpu3 tap/device found: 0x100854b1 (mfg: 0x258 (ARC International), part: 0x0085, ver: 0x1)
Info : JTAG tap: arc-em.cpu2 tap/device found: 0x100454b1 (mfg: 0x258 (ARC International), part: 0x0045, ver: 0x1)
Info : JTAG tap: arc-em.cpu1 tap/device found: 0x100054b1 (mfg: 0x258 (ARC International), part: 0x0005, ver: 0x1)
Info : starting gdb server for arc-em.cpu4 on 49101
Info : Listening on port 49101 for gdb connections
Info : starting gdb server for arc-em.cpu3 on 49102
Info : Listening on port 49102 for gdb connections
Info : starting gdb server for arc-em.cpu2 on 49103
Info : Listening on port 49103 for gdb connections
Info : starting gdb server for arc-em.cpu1 on 49104
Info : Listening on port 49104 for gdb connections
```

Then connect to the server using GDB (49104 port is used below in
GDB session, because cores are numbered in reverse order and 49104 port
corresponds to the first core):

```text
$ arc-elf32-gdb -quiet main.elf
(gdb) target remote :49104

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

Follow [the corresponding guide](../../platforms/board-hsdk-4xd.md#connecting-to-the-serial-terminal)
for ARC HS Development Kit.
