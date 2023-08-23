# 🕥 EM Starter Kit

!!! warning

    EM Starter Kit board is no longer supported. There is no guarantee that this guide will
    be applicable for the latest tools.

!!! info

    * Please refer to [board's documentation](../../platforms/board-emsk.md)
      for detailed information about how to setup the board for initial operation.
    * Refer [Getting OpenOCD](../../platforms/get-openocd.md) and [Using OpenOCD](../../platforms/use-openocd.md)
      for details about installing and using OpenOCD.

## Building an Application

Suppose, that EMSK 2.2 is used and EM7D core is selected.
Consider a simple application `main.c`:

```c
int main()
{
    int a = 1;
    int b = 2;
    int c = a + b;
    return c;
}
```

Create a [custom memory map](./memory.md) file with name `memory.x`
(memory maps for all EM Starter Kit version may be found in [toolchain's repository](https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain/tree/arc-releases/extras/dev_systems)):

```text
MEMORY
{
    ICCM : ORIGIN = 0x00000000, LENGTH = 256K
    DRAM : ORIGIN = 0x10000000, LENGTH = 128M
    DCCM : ORIGIN = 0x80000000, LENGTH = 128K
}

REGION_ALIAS("startup", ICCM)
REGION_ALIAS("text", ICCM)
REGION_ALIAS("data", DRAM)
REGION_ALIAS("sdata", DRAM)

PROVIDE (__stack_top = (0x17FFFFFF & -4) );
PROVIDE (__end_heap = (0x17FFFFFF) );
```


Build an application with support of UART:

```shell
arc-elf32-gcc -mcpu=em4_dmips -specs=emsk.specs -Wl,-marcv2elfx main.c -o main.elf
```

Or build without support of UART:

```shell
arc-elf32-gcc -mcpu=em4_dmips -specs=nosys.specs -Wl,-marcv2elfx main.c -o main.elf
```

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
