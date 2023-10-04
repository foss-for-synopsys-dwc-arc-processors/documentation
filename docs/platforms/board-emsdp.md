# ARC EM Software Development Platform

![ARC EM Software Development Platform](images/board-emsdp.jpg)

## Overview

The DesignWare® ARC® EM Software Development Platform (SDP) is a flexible
platform for rapid software development on ARC EM processors and subsystems.
It is intended to accelerate software development and debug of ARC EM
processor-based systems for a wide range of ultra-low power embedded
applications such as IoT, sensor fusion, and voice applications.

The EM SDP includes an FPGA-based hardware board with commonly used peripherals
and interfaces for extensibility. Downloadable platform packages containing
different hardware configurations enable the board to be programmed with
different ARC EM processors and subsystems. The packages also contain the
necessary software configuration information for the toolchain and embARC Open
Software Platform. The development platform is supported by Synopsys’
ARC MetaWare Development Toolkit, which includes a compiler, debugger and
libraries optimized for maximum performance with minimal code size. The embARC
Open Software Platform (OSP), available online from embARC.org, gives
developers online access to device drivers, FreeRTOS, middleware and examples
that enables them to quickly start software development for their ARC-based
embedded systems.

Key Features

* Xilinx Kintex-7 XC7K325T-2FBG676C
* 32 MByte Quad-SPI Flash memory (for configuration and operation)
* USB-JTAG bridge FT2232H
* FPGA configuration through:
    * JTAG
    * SPI Flash memory
* SPI Flash configuration through:
    * JTAG
    * USB
* Connectors
    * Arduino compatible pin headers
    * MicroBUS compatible pin headers
    * 3x Pmod compatible pin headers
    * 50 pin header 2.54mm (40 single-ended IO, 20 differential lanes, variable VCCIO)
    * Mictor Debug connector
    * 10 pin Debug connector 2mm
* 2 x 8 MByte PSRAM IS66WVC4M16EALL
* 32 MByte User Quad-SPI Flash memory
* Micro SDcard Socket
* Wireless module RS9113-NBZ-D1C (Wi-Fi/BT/BLE)
* 3-axis gyroscope, 3-axis accelerometer, 3-axis magnetometer MPU-9250
* Stereo Audio Codec MAX9880A
* 2 x PDM Microphones SPK0641HT4H-1
* 2 x 3.5mm RCA audio jacks (input/output)
* 100MHz User Clock Oscillator SiT8008
* Status LEDs
* Power LED
* 8 x User LEDs
* Reset Buttons
* User Button
* 2 x 4bit User DIP switches
* 12V Power Supply
* 12V fan

## Block Diagram

![ARC EM SDP block diagram](./images/board-emsdp-blocks.jpg)

## Connecting to the Serial Terminal

!!! warning

    On Linux machines it may be necessary for a user to be in `dialout`
    group to successfully connect to a serial terminal. In case of
    "Permission denied" error try to add a user to the group:

    ```shell
    sudo usermod -aG dialout username
    ```

Connecting to the board using USB data port allows to connect to the serial
terminal over UART. You need to configure these parameters of a serial
terminal to interact with the serial port:

* baud-rate 115200
* 8 data bits
* 1 stop Bit

On Windows [Putty](https://www.putty.org/) or any similar software may be used for connecting
to the serial terminal. You can find the port number in **Device Manager** in
**Ports (COM & LPT)** section: **USB Serial Port (COMx)** where **COMx** is
a value for **Serial line** field in Putty's. Other parameters may be set
**Connection → Serial** menu.

On Linux `minicom` or other similar utilities may be used. Here is an example
of command line for `minicom`:

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

## How to Manually Flash a Firmware

You can flash a firmware of ARC CPU manually using [Digilent Adept utilities](./digilent.md).
Firstly, ensure that EM SDP in connected to the host:

```text
$ djtgcfg enum
Found 1 device(s)

Device: JtagSmt1
    Device Transport Type: 00020001 (USB)
    Product Name:          Digilent JTAG-SMT1
    User Name:             JtagSmt1
    Serial Number:         210203826102
```

Device name for EM SDP is `JtagSmt1`. Then get a firmware from EM SDP bundle
(for example, `emsdp_em11d_dfss.bit` for EM11D), init the board using
`init` command and program board's FPGA:

```text
$ djtgcfg -d JtagSmt1 init
Initializing scan chain...
Found Device ID: 43651093

Found 1 device(s):
    Device 0: XC7K325T

$ djtgcfg -d JtagSmt1 prog -i 0 -f emsdp_em11d_dfss.bit
Programming device. Do not touch your board. This may take a few minutes...
Programming succeeded.
```

## OpenOCD Configuration File

!!! info

    Also refer [Getting OpenOCD](./get-openocd.md) and 
    [Using OpenOCD](./use-openocd.md) for details about installing
    and using OpenOCD.

There is no a configuration file for EM SDP in OpenOCD yet. You can create it
manually and pass it to OpenOCD as a regular configuration file using `-f`
parameter:

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

## Useful Links

* [ARC EM Software Development Platform - User Guide](files/ARC_EM_SDP_User_Guide.pdf)
* [Official Synopsys Page](https://www.synopsys.com/dw/ipdir.php?ds=arc-em-software-development-platform)
* [embARC Open Software Platform Documentation](https://foss-for-synopsys-dwc-arc-processors.github.io/embarc_osp)
* [embARC Open Software Platform Releases Page](https://github.com/foss-for-synopsys-dwc-arc-processors/embarc_osp/releases)

## How to Order

* [Order EM Software Development Platform on Trenz Electronic](https://shop.trenz-electronic.de/en/Products/Synopsys/)

## Support

* [Ask a question, report a bug or request an enhancement](https://github.com/foss-for-synopsys-dwc-arc-processors/ARC-Development-Systems-Forum/wiki/Reporting-a-bug)
