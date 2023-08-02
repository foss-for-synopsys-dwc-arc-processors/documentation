# Building Applications and Debugging on Development Platforms

## Preface

* Refer to [Development Platforms Table](#development-platforms-table) section
  for options for all available configurations of the board.
* Follow [Memory Maps and Linker Scripts](./memory.md) guide for details
  about `memory.x` files and where they may be obtained.
* Follow a [corresponding manual](../../platforms/get-openocd.md) to obtain
  and configure OpenOCD.

## Connecting Using UART

Connecting a board to the host computer allows you using a serial port terminal for interacting with the board.
You can use any software to connect to the serial port terminal:

* On Windows you can use HyperTerminal or Putty.
* On Linux you can use `minicom`, `gtkterm`, `cutecom`, etc.

You can find detailed instructions for particular boards in corresponding sections.

## EM Starter Kit

!!! info

    Please refer to [board's documentation](https://github.com/foss-for-synopsys-dwc-arc-processors/ARC-Development-Systems-Forum/wiki/ARC-Development-Systems-Forum-Wiki-Home#arc-em-starter-kit-1)
    for detailed information about how to setup the board for initial operation.

### Building an Application

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

Build an application with support of UART:

```shell
$ cp <path-to-toolchain>/arc-*/lib/emsk2.2_em9d.x memory.x
$ arc-elf32-gcc -mcpu=em4_dmips -specs=emsk.specs main.c -o main.elf
```

Build without support of UART:

```shell
$ cp <path-to-toolchain>/arc-*/lib/emsk2.2_em9d.x memory.x
$ arc-elf32-gcc -mcpu=em4_dmips -specs=nosys.specs -Wl,-marcv2elfx main.c -o main.elf
```

### Running an Application

Follow [Running OpenOCD Server](#running-openocd-server) section and start OpenOCD
with 49101 port and `snps_em_sk_v2.2.cfg` configuration file. Here is
a possible output:

```text
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

### Connecting Using UART

If you are going to use input/output, then you need to configure these
parameters of a serial terminal to interact with the serial port:

* baud-rate 115200
* 8 data bits
* 1 stop Bit

For `minicom` use this command:

```shell
$ minicom -8 -b 115200 -D /dev/ttyUSB1
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

## HS Development Kit

!!! info

    Please refer to board's documentation for detailed information about how to setup the board for initial operation:

    * [ARC HS Development Kit](https://github.com/foss-for-synopsys-dwc-arc-processors/ARC-Development-Systems-Forum/wiki/ARC-Development-Systems-Forum-Wiki-Home#arc-hs-development-kit-1)
    * [ARC HS4x/HS4xD Development Kit](https://github.com/foss-for-synopsys-dwc-arc-processors/ARC-Development-Systems-Forum/wiki/ARC-Development-Systems-Forum-Wiki-Home#arc-hs4xhs4xd-development-kit-1)

### Building an Applications

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

Build an application with support of UART:

```shell
$ cp <path-to-toolchain>/arc-*/lib/hsdk.x memory.x
$ arc-elf32-gcc -mcpu=hs38_linux -specs=hsdk.specs main.c -o main.elf
```

Build without support of UART, but if you are going to use interrupts (it allows
to catch memory errors):

```shell
$ cp <path-to-toolchain>/arc-*/lib/hsdk.x memory.x
$ arc-elf32-gcc -mcpu=hs38_linux -specs=nosys.specs -Wl,-marcv2elfx \
                -Wl,--defsym=ivtbase_addr=0x90000000 main.c -o main.elf
```

A simple build if you are not going to use UART and interrupts:

```shell
$ arc-elf32-gcc -mcpu=hs38_linux -specs=nosys.specs main.c -o main.elf
```

### Running an Application

Follow [Running OpenOCD Server](#running-openocd-server) section and start OpenOCD
with 49101 port and `snps_hsdk.cfg` (for HSDK) of `snps_hsdk_4xd.cfg`
(for HSDK 4xD) configuration file. Here is a possible output for HSDK 4xD:

```text
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

### Connecting Using UART

If you are going to use input/output, then you need to configure these
parameters of a serial terminal to interact with the serial port:

* baud-rate 115200
* 8 data bits
* 1 stop Bit
* No HW/SW flow control

For `minicom` use this command:

```shell
$ minicom -8 -b 115200 -D /dev/ttyUSB0 -s
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

## Development Platforms Table

| Board       | Core       | `-specs=`     | Memory map        | OpenOCD config        | GCC options                                                                  |
|-------------|------------|---------------|-------------------|-----------------------|------------------------------------------------------------------------------|
| EMSK 1      | EM4        | `emsk.specs`  | `emsk1_em4.x`     | `snps_em_sk_v1.cfg`   | `-mcpu=em4_dmips -mmpy-option=wlh5`                                          |
| EMSK 1      | EM6        | `emsk.specs`  | `emsk1_em6.x`     | `snps_em_sk_v1.cfg`   | `-mcpu=em4_dmips -mmpy-option=wlh5`                                          |
| EMSK 2.0    | EM5D       | `emsk.specs`  | `emsk2.1_em5d.x`  | `snps_em_sk_v2.1.cfg` | `-mcpu=em4 -mswap -mnorm -mmpy-option=wlh3 -mbarrel-shifter`                 |
| EMSK 2.0    | EM7D       | `emsk.specs`  | `emsk2.1_em7d.x`  | `snps_em_sk_v2.1.cfg` | `-mcpu=em4 -mswap -mnorm -mmpy-option=wlh3 -mbarrel-shifter`                 |
| EMSK 2.0    | EM7DFPU    | `emsk.specs`  | `emsk2.1_em7d.x`  | `snps_em_sk_v2.1.cfg` | `-mcpu=em4 -mswap -mnorm -mmpy-option=wlh3 -mbarrel-shifter -mfpu=fpuda_all` |
| EMSK 2.1    | EM5D       | `emsk.specs`  | `emsk2.1_em5d.x`  | `snps_em_sk_v2.1.cfg` | `-mcpu=em4_dmips -mmpy-option=wlh3`                                          |
| EMSK 2.1    | EM7D       | `emsk.specs`  | `emsk2.1_em7d.x`  | `snps_em_sk_v2.1.cfg` | `-mcpu=em4_dmips -mmpy-option=wlh3`                                          |
| EMSK 2.1    | EM7DFPU    | `emsk.specs`  | `emsk2.1_em7d.x`  | `snps_em_sk_v2.1.cfg` | `-mcpu=em4_fpuda -mmpy-option=wlh3`                                          |
| EMSK 2.2    | EM7D       | `emsk.specs`  | `emsk2.2_em9d.x`  | `snps_em_sk_v2.2.cfg` | `-mcpu=em4_dmips`                                                            |
| EMSK 2.2    | EM9D       | `emsk.specs`  | `emsk2.2_em9d.x`  | `snps_em_sk_v2.2.cfg` | `-mcpu=em4_fpus -mfpu=fpus_all`                                              |
| EMSK 2.2    | EM11D      | `emsk.specs`  | `emsk2.2_em11d.x` | `snps_em_sk_v2.2.cfg` | `-mcpu=em4_fpuda -mfpu=fpuda_all`                                            |
| EMSK 2.3    | EM7D       | `emsk.specs`  | `emsk2.2_em9d.x`  | `snps_em_sk_v2.3.cfg` | `-mcpu=em4_dmips`                                                            |
| EMSK 2.3    | EM9D       | `emsk.specs`  | `emsk2.2_em9d.x`  | `snps_em_sk_v2.3.cfg` | `-mcpu=em4_fpus -mfpu=fpus_all`                                              |
| EMSK 2.3    | EM11D      | `emsk.specs`  | `emsk2.2_em11d.x` | `snps_em_sk_v2.3.cfg` | `-mcpu=em4_fpuda -mfpu=fpuda_all`                                            |
| HSDK        | HS3x       | `hsdk.specs`  | `hsdk.x`          | `snps_hsdk.cfg`       | `-mcpu=hs38_linux`                                                           |
| HSDK 4x/4xD | HS4x/HS4xD | `hsdk.specs`  | `hsdk.x`          | `snps_hsdk_4xd.cfg`   | `-mcpu=hs38_linux`                                                           |
| IoTDK       | ???        | `iotdk.specs` | `iotdk.x`         | `snps_iotdk.cfg`      | `-mcpu=em4_dmips`                                                            |

## Running OpenOCD Server

OpenOCD is used for starting a GDB server for connecting to a board.
Suppose, that EM Starter Kit 2.3 is used. If you’ve downloaded IDE bundle for
Linux then you can run OpenOCD this way (replace `<ide>` by a path to the
directory of IDE bundle):

```shell
$ <ide>/bin/openocd -s <ide>/share/openocd/scripts -c 'gdb_port 49101' -f board/snps_em_sk_v2.3.cfg
```

If you’ve built and installed OpenOCD manually then you can run OpenOCD this way:

```shell
$ openocd  -c 'gdb_port 49101' -f board/snps_em_sk_v2.2.cfg
```

If you’ve downloaded and installed IDE bundle for Windows then you can run OpenOCD this way:

```shell
$ openocd -s C:\arc_gnu\share\openocd\scripts -c "gdb_port 49101" -f board\snps_em_sk_v2.3.cfg
```
