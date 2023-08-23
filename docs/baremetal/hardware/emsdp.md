# EM Software Development Platform

!!! info

    * Please refer to [board's documentation](../../platforms/board-emsdp.md)
      for detailed information about how to setup the board for initial operation.
    * Refer [Getting OpenOCD](../../platforms/get-openocd.md) and [Using OpenOCD](../../platforms/use-openocd.md)
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

In this guide we use EM11D core as an example.

## Building an Application

Consider a simple application with name `main.c`:

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
    ICCM0 : ORIGIN = 0x10000000, LENGTH = 128K
    PSRAM : ORIGIN = 0x40000000, LENGTH =  16M
    DCCM0 : ORIGIN = 0x80000000, LENGTH = 128K
}

REGION_ALIAS("startup", ICCM0)
REGION_ALIAS("text", ICCM0)
REGION_ALIAS("data", DCCM0)
REGION_ALIAS("sdata", DCCM0)
```

Build the application with support of UART:

```text
arc-elf32-gcc -mcpu=em4_fpuda -mmpy-option=6 -mfpu=fpuda_all -specs=emsdp.specs \
              -Wl,-marcv2elfx -Wl,--defsym=ivtbase_addr=0x10000000 \
              main.c -o main.elf
```

Or build the application without support of UART:

```text
arc-elf32-gcc -mcpu=em4_fpuda -mmpy-option=6 -mfpu=fpuda_all -specs=nosys.specs \
              -Wl,-marcv2elfx -Wl,--defsym=ivtbase_addr=0x10000000 \
              main.c -o main.elf
```

## Running an Application

Create a configuration file for OpenOCD and name it `emsdp.cfg`. It's necessary
because it's not included in OpenOCD distribution yet:

```tcl
# Configure JTAG cable
# EM SDP has built-in FT2232 chip, which is similar to Digilent HS-1.
adapter driver ftdi

# Only specify FTDI serial number if it is specified via
# "set _ZEPHYR_BOARD_SERIAL 12345" before reading this script
if { [info exists _ZEPHYR_BOARD_SERIAL] } {
       ftdi_serial $_ZEPHYR_BOARD_SERIAL
}

ftdi vid_pid 0x0403 0x6010
ftdi layout_init 0x0088 0x008b
ftdi channel 0

# EM11D requires 10 MHz.
adapter speed 10000

# ARCs support only JTAG.
transport select jtag

source [find cpu/arc/em.tcl]

set _CHIPNAME arc-em
set _TARGETNAME $_CHIPNAME.cpu

# EM SDP IDENTITY is 0x200444b1
jtag newtap $_CHIPNAME cpu -irlen 4 -ircapture 0x1 -expected-id 0x200044b1

set _coreid 0
set _dbgbase [expr {0x00000000 | ($_coreid << 13)}]

target create $_TARGETNAME arcv2 -chain-position $_TARGETNAME \
  -coreid 0 -dbgbase $_dbgbase -endian little

# There is no SRST, so do a software reset
$_TARGETNAME configure -event reset-assert "arc_em_reset $_TARGETNAME"

arc_em_init_regs
```

Start OpenOCD using `emsdp.cfg` configuration file:

```text
$ openocd -f ./emsdp.cfg
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

After resetting the EM SDP you will see this output of the bootloader:

```text
U-Boot 2020.04 (Sep 09 2022 - 08:54:58 +0200)

CPU:   ARC EM11D v5.6 at 40 MHz
Subsys:ARC Data Fusion IP Subsystem
Model: snps,emsdp
Board: ARC EM Software Development Platform v1.2
DRAM:  16 MiB
PSRAM initialized.
MMC:   mmc0@f0010000: 0
Loading Environment from FAT... MMC: no card present
In:    serial0@f0004000
Out:   serial0@f0004000
Err:   serial0@f0004000
emsdp#
```
