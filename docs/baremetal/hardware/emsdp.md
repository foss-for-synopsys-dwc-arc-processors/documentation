# EM Software Development Platform

!!! info

    Please refer to [board's documentation](../../platforms/board-emsdp.md)
    for detailed information about how to setup the board for initial operation.
    
    Refer [Getting OpenOCD](../../platforms/get-openocd.md) and [Using OpenOCD](../../platforms/use-openocd.md)
    for details about installing and using OpenOCD.

## Preface

EM SDP board is shipped with with FPGA chip. There is a number of ARC EM images
available for writing to it. Here is a list of those images and corresponding
target options for GCC:

| CPU image | GCC targets options                                               |
|-----------|-------------------------------------------------------------------|
| EM11D     | `-mcpu=em4_fpuda -mmpy-option=6 -mfpu=fpuda_all`                  |
| EM9D      | `-mcpu=em4_fpuda -mmpy-option=6 -mfpu=fpuda_all`                  |
| EM7D      | `-mcpu=em4_dmips -mno-div-rem -mmpy-option=3 -mfpu=fpuda_all`     |
| EM7D ESP  | `-mcpu=em4_fpuda -mmpy-option=6 -mbarrel-shifter -mfpu=fpuda_all` |
| EM6       | `-mcpu=em4_dmips -mno-div-rem -mmpy-option=3 -mfpu=fpuda_all`     |
| EM5D      | `-mcpu=em4_fpuda -mmpy-option=6 -mfpu=fpuda_all`                  |
| EM4       | `-mcpu=em4_fpuda -mno-div-rem -mmpy-option=3 -mfpu=fpuda_all`     |

In this guide we use EM11D core as an example. Also, the version of the EM SDP
firmware is considered to be 1.0 or 1.1.

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
arc-elf32-gcc -mcpu=em4_fpuda -mmpy-option=6 -mfpu=fpuda_all \
              -specs=emsdp1.1.specs main.c -o main.elf
```

`-specs=emsdp1.1.specs` sets a proper [memory map](../general/memory.md) and links the
application with additional startup code and UART library for input/output
operations.

Here is a list of all available `specs` files:

| Specs file           | EM SDP firmware | Description                               |
|----------------------|-----------------|-------------------------------------------|
| `emsdp1.1.specs`     | 1.0, 1.1        | Code and data are placed in ICCM and DCCM |
| `emsdp1.1_ram.specs` | 1.0, 1.1        | Code and data are placed in RAM           |
| `emsdp1.2.specs`[^1] | 1.2             | Code and data are placed in ICCM and DCCM |
| `emsdp1.2_ram.specs` | 1.2             | Code and data are placed in RAM           |

[^1]:
    The memory map for EM SDP 1.2 firmware differs from the memory map for
    EM SDP 1.0 and 1.1. That is why there are different specs files.

## Running an Application

Follow [Using OpenOCD](../../platforms/use-openocd.md) guide and start OpenOCD
with `snps_em_sk_v2.3.cfg` configuration file. Here is a possible output:

```text
$ openocd  -f board/snps_em_sk_v2.3.cfg
Open On-Chip Debugger 0.12.0+dev-gffa52f0e0 (2023-08-02-10:41)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : ftdi: if you experience problems at higher adapter clocks, try the command "ftdi tdo_sample_edge falling"
Info : clock speed 10000 kHz
Info : JTAG tap: arc-em.cpu tap/device found: 0x200044b1 (mfg: 0x258 (ARC International), part: 0x0004, ver: 0x2)
Info : starting gdb server for arc-em.cpu on 3333
Info : Listening on port 3333 for gdb connections
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

Follow [the corresponding guide](../../platforms/board-emsdp.md#connecting-to-the-serial-terminal)
for ARC EM Software Development Platform.
