# HS Development Kit

!!! info

    Please refer to board's documentation for detailed information about how to
    setup the board for initial operation:

    * [ARC HS Development Kit](../../platforms/board-hsdk.md)
    * [ARC HS4x/HS4xD Development Kit](../../platforms/board-hsdk-4xd.md)

    Also refer [Getting OpenOCD](../../platforms/get-openocd.md) and 
    [Using OpenOCD](../../platforms/use-openocd.md) for details about installing
    and using OpenOCD.

## Building an Application

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

Create a [custom memory map](./memory.md) file with name `memory.x`:

```text
MEMORY
{
    DRAM : ORIGIN = 0x90000000, LENGTH = 0x50000000
}

REGION_ALIAS("startup", DRAM)
REGION_ALIAS("text", DRAM)
REGION_ALIAS("data", DRAM)
REGION_ALIAS("sdata", DRAM)
```

Build an application with support of UART:

```shell
arc-elf32-gcc -mcpu=hs38_linux -specs=hsdk.specs -Wl,-marcv2elfx \
              -Wl,--defsym=ivtbase_addr=0x90000000 \
              main.c -o main.elf
```

Or build without support of UART, but if you are going to use interrupts (it allows
to catch memory errors):

```shell
arc-elf32-gcc -mcpu=hs38_linux -specs=nosys.specs -Wl,-marcv2elfx \
              -Wl,--defsym=ivtbase_addr=0x90000000 \
              main.c -o main.elf
```

Here is a simple command line if you are not going to use UART and interrupts:

```shell
arc-elf32-gcc -mcpu=hs38_linux -specs=nosys.specs main.c -o main.elf
```

## Running an Application

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

## Connecting Using UART

If you are going to use input/output, then you need to configure these
parameters of a serial terminal to interact with the serial port:

* baud-rate 115200
* 8 data bits
* 1 stop Bit
* No HW/SW flow control

For `minicom` use this command:

```shell
minicom -8 -b 115200 -D /dev/ttyUSB0 -s
```

Then choose `Serial port setup`, press `F` to disable `Hardware Flow Control`, press `Enter` key
and then choose `Exit` to close the configuration menu.

After resetting HSDK you will see this output of the bootloader:

```text
********************************
**       Synopsys, Inc.       **
**   ARC HS Development Kit   **
********************************
** IC revision: Rev 2.0
** Bootloader verbosity: Normal
** Starting HS Core 1
** HS Core running @ 500 MHz
fptr = 8** HS Core fetching application from SPI flash
** HS Core starting application
<debug_uart> 

U-Boot 2020.01 (Apr 26 2020 - 22:30:20 +0300)

CPU:   ARC HS v4.0 at 500 MHz
Model: snps,hsdk-4xd
Board: Synopsys ARC HS4x/HS4xD Development Kit
DRAM:  1 GiB
Relocation Offset is: 3ef8a000
MMC:   mmc0@f000a000: 0
Loading Environment from FAT... MMC: no card present
In:    serial0@f0005000
Out:   serial0@f0005000
Err:   serial0@f0005000
Clock values are saved to environment
Net:   
Warning: ethernet@f0008000 (eth0) using random MAC address - c2:26:b0:99:98:4a
eth0: ethernet@f0008000
hsdk-4xd# 
```
