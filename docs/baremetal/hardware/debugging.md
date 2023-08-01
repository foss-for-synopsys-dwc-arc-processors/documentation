# Building Applications and Debugging on Development Platforms

## Getting Started with OpenOCD

Follow a [corresponding manual](../../platforms/openocd.md) to obtain and configure OpenOCD.

## Connecting Using UART

Connecting a board to the host computer allows you using a serial port terminal for interacting with the board.
You can use any software to connect to the serial port terminal:

* On Windows you can use HyperTerminal or Putty.
* On Linux you can use `minicom`, `gtkterm`, `cutecom`, etc.

You can find detailed instructions for particular boards in corresponding sections.

## EM Starter Kit

### About the Board

Please refer to board's documentation for detailed information about how to setup the board for initial operation:

* [EM Starter Kit 2.2/2.3](https://github.com/foss-for-synopsys-dwc-arc-processors/ARC-Development-Systems-Forum/wiki/ARC-Development-Systems-Forum-Wiki-Home#arc-em-starter-kit-1)

### Build Applications

!!! info

    Refer to [Development Platforms Table](#development-platforms-table) section
    for options for all available configurations of the board.

Suppose, that EMSK 2.2/2.3 is used and EM7D core is selected.

Build an application with support of UART:

    cp <path-to-toolchain>/arc-*/lib/emsk2.2_em9d.x memory.x
    arc-elf32-gcc -mcpu=em4_dmips -specs=emsk.specs main.c -o main.elf

Build without support of UART:

    cp <path-to-toolchain>/arc-*/lib/emsk2.2_em9d.x memory.x
    arc-elf32-gcc -mcpu=em4_dmips -specs=nosys.specs -Wl,-marcv2elfx main.c -o main.elf

### Connecting Using UART

You need to configure these parameters to interact with the serial port:

* baud-rate 115200
* 8 data bits
* 1 stop Bit

For `minicom` use this command:

    minicom -8 -b 115200 -D /dev/ttyUSB1

## HS Development Kit

### About the Board

HS Development Kit is based on a custom designed Synopsys ARC SoC containing the ARC HS38x4 (quad core) processor. Please refer to board's documentation for detailed information about how to setup the board for initial operation:

* [ARC HS Development Kit](https://github.com/foss-for-synopsys-dwc-arc-processors/ARC-Development-Systems-Forum/wiki/ARC-Development-Systems-Forum-Wiki-Home#arc-hs-development-kit-1)
* [ARC HS4x/HS4xD Development Kit](https://github.com/foss-for-synopsys-dwc-arc-processors/ARC-Development-Systems-Forum/wiki/ARC-Development-Systems-Forum-Wiki-Home#arc-hs4xhs4xd-development-kit-1)

### Build Applications

> ℹ️ Refer to [Development Platforms Table](#development-platforms-table) section for
> for options for all available configurations of the board.

Build an application with support of UART:

    cp <path-to-toolchain>/arc-*/lib/hsdk.x memory.x
    arc-elf32-gcc -mcpu=hs38_linux -specs=hsdk.specs main.c -o main.elf

Build without support of UART, but if you are going to use interrupts (it allows
to catch memory errors):

    cp <path-to-toolchain>/arc-*/lib/hsdk.x memory.x
    arc-elf32-gcc -mcpu=hs38_linux -specs=nosys.specs -Wl,-marcv2elfx \
                  -Wl,--defsym=ivtbase_addr=0x90000000 main.c -o main.elf

A simple build if you are not going to use UART and interrupts:

    arc-elf32-gcc -mcpu=hs38_linux -specs=nosys.specs main.c -o main.elf

### Connecting Using UART

You need to configure these parameters to interact with the serial port:

* baud-rate 115200
* 8 data bits
* 1 stop Bit
* No HW/SW flow control

For `minicom` use this command:

    minicom -8 -b 115200 -D /dev/ttyUSB0 -s

Then choose `Serial port setup`, press `F` to disable `Hardware Flow Control`, press `Enter` key
and then choose `Exit` to close the configuration menu.

After resetting HSDK you will see this output of the bootloader:

```
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

| Board | Core | `-specs=` | Memory map | GCC options |
| --- | --- | --- | --- | --- |
| EMSK 1 | EM4 | `emsk.specs` | `emsk1_em4.x` | `-mcpu=em4_dmips -mmpy-option=wlh5` |
| EMSK 1 | EM6 | `emsk.specs` | `emsk1_em6.x` | `-mcpu=em4_dmips -mmpy-option=wlh5` |
| EMSK 2.0 | EM5D | `emsk.specs` | `emsk2.1_em5d.x` | `-mcpu=em4 -mswap -mnorm -mmpy-option=wlh3 -mbarrel-shifter` |
| EMSK 2.0 | EM7D | `emsk.specs` | `emsk2.1_em7d.x` | `-mcpu=em4 -mswap -mnorm -mmpy-option=wlh3 -mbarrel-shifter` |
| EMSK 2.0 | EM7DFPU | `emsk.specs` | `emsk2.1_em7d.x` | `-mcpu=em4 -mswap -mnorm -mmpy-option=wlh3 -mbarrel-shifter -mfpu=fpuda_all` |
| EMSK 2.1 | EM5D | `emsk.specs` | `emsk2.1_em5d.x` | `-mcpu=em4_dmips -mmpy-option=wlh3` |
| EMSK 2.1 | EM7D | `emsk.specs` | `emsk2.1_em7d.x` | `-mcpu=em4_dmips -mmpy-option=wlh3` |
| EMSK 2.1 | EM7DFPU | `emsk.specs` | `emsk2.1_em7d.x` | `-mcpu=em4_fpuda -mmpy-option=wlh3` |
| EMSK 2.2 | EM7D | `emsk.specs` | `emsk2.2_em9d.x` | `-mcpu=em4_dmips` |
| EMSK 2.2 | EM9D | `emsk.specs` | `emsk2.2_em9d.x` | `-mcpu=em4_fpus -mfpu=fpus_all` |
| EMSK 2.2 | EM11D | `emsk.specs` | `emsk2.2_em11d.x` | `-mcpu=em4_fpuda -mfpu=fpuda_all` |
| EMSK 2.3 | EM7D | `emsk.specs` | `emsk2.2_em9d.x` | `-mcpu=em4_dmips` |
| EMSK 2.3 | EM9D | `emsk.specs` | `emsk2.2_em9d.x` | `-mcpu=em4_fpus -mfpu=fpus_all` |
| EMSK 2.3 | EM11D | `emsk.specs` | `emsk2.2_em11d.x` | `-mcpu=em4_fpuda -mfpu=fpuda_all` |
| HSDK | HS3x | `hsdk.specs` | `hsdk.x` | `-mcpu=hs38_linux` |
| HSDK 4x/4xD | HS4x/HS4xD | `hsdk.specs` | `hsdk.x` | `-mcpu=hs38_linux` |
| IoTDK | ??? | `iotdk.specs` | `iotdk.x` | `-mcpu=em4_dmips` |

## Debugging Applications Using OpenOCD

| Board | OpenOCD configuration |
| --- | --- |
| EMSK 1 | `snps_em_sk_v1.cfg` |
| EMSK 2.0 | `snps_em_sk_v2.1.cfg` |
| EMSK 2.1 | `snps_em_sk_v2.1.cfg` |
| EMSK 2.2 | `snps_em_sk_v2.2.cfg` |
| EMSK 2.3 | `snps_em_sk_v2.3.cfg` |
| HSDK | `snps_hsdk.cfg` |
| HSDK 4x/4xD | `snps_hsdk.cfg` |
| IoTDK | `snps_iotdk.cfg` |