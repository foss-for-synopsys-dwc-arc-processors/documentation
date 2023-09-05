# 🕥 EM Starter Kit

!!! warning

    EM Starter Kit board is no longer supported. There is no guarantee that this guide will
    be applicable for the latest tools.

!!! info

    * Please refer to [board's documentation](../../platforms/board-emsk.md)
      for detailed information about how to setup the board for initial operation.
    * Refer [Getting OpenOCD](../../platforms/get-openocd.md) and [Using OpenOCD](../../platforms/use-openocd.md)
      for details about installing and using OpenOCD.

## Preface

EM Starter Kit board is shipped with with FPGA chip. There is a number of firmwares
available for writing to it. Also, you can select one of the cores presented in the firmware.
Here is a list of those firmwares and cores and corresponding target options for GCC:

| Firmware | Core    | GCC targets options                                                          |
|----------|---------|------------------------------------------------------------------------------|
| 2.2, 2.3 | EM11D   | `-mcpu=em4_fpuda -mfpu=fpuda_all`                                            |
| 2.2, 2.3 | EM9D    | `-mcpu=em4_fpus -mfpu=fpus_all`                                              |
| 2.2, 2.3 | EM7D    | `-mcpu=em4_dmips`                                                            |
| 2.1      | EM7DFPU | `-mcpu=em4_fpuda -mmpy-option=wlh3`                                          |
| 2.1      | EM7D    | `-mcpu=em4_dmips -mmpy-option=wlh3`                                          |
| 2.1      | EM5D    | `-mcpu=em4_dmips -mmpy-option=wlh3`                                          |
| 2.0      | EM7DFPU | `-mcpu=em4 -mswap -mnorm -mmpy-option=wlh3 -mbarrel-shifter -mfpu=fpuda_all` |
| 2.0      | EM7D    | `-mcpu=em4 -mswap -mnorm -mmpy-option=wlh3 -mbarrel-shifter`                 |
| 2.0      | EM5D    | `-mcpu=em4 -mswap -mnorm -mmpy-option=wlh3 -mbarrel-shifter`                 |
| 1.0      | EM6     | `-mcpu=em4_dmips -mmpy-option=wlh5`                                          |
| 1.0      | EM4     | `-mcpu=em4_dmips -mmpy-option=wlh5`                                          |

In this guide we use EM7D core as an example. Also, the version of the EMSK
firmware is considered to be 2.2 or 2.3.

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
arc-elf32-gcc -mcpu=em4_dmips -specs=emsk2.2_em7d.specs main.c -o main.elf
```

`-specs=emsk2.2_em7d.specs` sets a proper [memory map](./memory.md) and links the
application with additional startup code and UART library for input/output
operations.

Here is a list of all available `specs` files:

| Specs file                | Firmware | Core  | Description                               |
|---------------------------|----------|-------|-------------------------------------------|
| `emsk2.2_em11d.specs`     | 2.2, 2.3 | EM11D | Code and data are placed in ICCM and DCCM |
| `emsk2.2_em11d_ram.specs` | 2.2, 2.3 | EM11D | Code and data are placed in RAM           |
| `emsk2.2_em9d.specs`      | 2.2, 2.3 | EM9D  | Code and data are placed in ICCM and DCCM |
| `emsk2.2_em9d_ram.specs`  | 2.2, 2.3 | EM9D  | Code and data are placed in RAM           |
| `emsk2.2_em7d.specs`      | 2.2, 2.3 | EM7D  | Code and data are placed in ICCM and DCCM |
| `emsk2.2_em7d_ram.specs`  | 2.2, 2.3 | EM7D  | Code and data are placed in RAM           |
| `emsk2.1_em7d.specs`      | 2.1      | EM7D  | Code and data are placed in ICCM and DCCM |
| `emsk2.1_em7d_ram.specs`  | 2.1      | EM7D  | Code and data are placed in RAM           |
| `emsk2.1_em5d.specs`      | 2.1      | EM5D  | Code and data are placed in ICCM and DCCM |
| `emsk1_em6.specs`         | 1.0      | EM6   | Code and data are placed in ICCM and DCCM |
| `emsk1_em6_ram.specs`     | 1.0      | EM6   | Code and data are placed in RAM           |
| `emsk1_em4.specs`         | 1.0      | EM4   | Code and data are placed in ICCM and DCCM |

## Running an Application

Follow [Using OpenOCD](../../platforms/use-openocd.md) guide and start OpenOCD
with 49101 port and `snps_em_sk_v2.2.cfg` configuration file. Here is
a possible output:

```text
$ openocd -f board/snps_em_sk_v2.2.cfg
Open On-Chip Debugger 0.9.0-dev (2023-05-21-06:23)
Licensed under GNU GPL v2
For bug reports, read
    http://openocd.sourceforge.net/doc/doxygen/bugs.html
adapter speed: 5000 kHz
Info : clock speed 5000 kHz
Info : JTAG tap: arc-em.cpu tap/device found: 0x200044b1 (mfg: 0x258, part: 0x0004, ver: 0x2)
Info : JTAG tap: arc-em.cpu tap/device found: 0x200044b1 (mfg: 0x258, part: 0x0004, ver: 0x2)
target state: halted
```

Then connect to the server using GDB:

```text
$ arc-elf32-gdb -quiet main.elf

# Connect. Replace 49101 with port of your choice if you changed it when starting OpenOCD
(gdb) target remote :49101

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

## Connecting Using UART

If you are going to use input/output, then you need to configure these
parameters of a serial terminal to interact with the serial port:

* baud-rate 115200
* 8 data bits
* 1 stop Bit

For `minicom` use this command:

```shell
minicom -8 -b 115200 -D /dev/ttyUSB1
```

After resetting the EMSK 2.2 you will see this output of the bootloader:

```text
***********************************
**       Synopsys, Inc.          **
**     ARC EM Starter kit        **
**                               **
** Comprehensive software stacks **
**   available from embARC.org   **
**                               **
***********************************
Firmware   Jan 11 2016, v2.2
Bootloader Dec 29 2015, v1.1
ARC EM11D, core configuration #3 

ARC IDENTITY = 0x42
RF_BUILD = 0xc902
TIMER_BUILD = 0x10304
ICCM_BUILD = 0x804
DCCM_BUILD = 0x10804
I_CACHE_BUILD = 0x135104
D_CACHE_BUILD = 0x215104

SelfTest PASSED

Info: No boot image found
```
