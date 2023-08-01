# Using OpenOCD

!!! warning

    This section is under construction.

## How to use OpenOCD on Linux

> Connection host is a host that is connected to the debug target via USB cable
> and runs OpenOCD. Debug host is a hsot that runs GDB, which connects to the
> OpenOCD with TCP connection. Ehily typically it is the same host, they
> actually can be different host and it is important to distinguish them.

Connect debug target to the connection host. AXS10x products and EM Starter Kit
have built-in debug cable, the don't require a separate Digilent HS cable. HS
is required only for other debug targets like ML-509 board, etc.

Run lsusb application to ensure that FTDI device is there:

    $ lsusb

In case of HS1, EM Starter Kit and AXS10x there should line like this:

    Bus 001 Device 002: ID 0403:6010 Future Technology Devices International, Ltd FT2232C Dual USB-UART/FIFO IC

In case if HS2 there should be line like this:

    Bus 001 Device 003: ID 0403:6014 Future Technology Devices International, Ltd FT232H Single HS USB-UART/FIFO IC

Note that exact lines could differ from host to host.

Run OpenOCD:

    $ openocd -f <openocd.cfg>

Valid openocd.cfg files are installed into
`/usr/local/share/openocd/scripts/board/`:

* snps_em_sk_v2.3.cfg - ARC EM Starter Kit verison 2.3
* snps_axs101.cfg - ARC AXS101 Software Development platform
* snps_axs103_hs36.cfg - ARC AXS103 in configuration with ARC HS36 core
* snps_hsdk.cfg - ARC HSDK

On the debug host (PC with GDB) start an ELF32 GDB debugger:

    $ arc-elf32-gdb ./<elf_app_to_debug>

Make the connection between arc-elf32-gdb and OpenOCD:

    (gdb) target remote <connection host ip address>:3333

Load image to be debugged (./<app_to_debug>.elf) into the target memory:

    (gdb) load

Set breakpoints at functions main and exit:

    (gdb) break main
    (gdb) break exit

Start the execution on target of the image to debug, to reach function main:

  (gdb) continue

Resume execution to reach function exit:

  (gdb) continue

## How to use OpenOCD on Windows

WinUSB driver should be used to allow OpenOCD to connect to debug cable and
that driver would replace FTDI proprietary drivers for Digilent cables or EM
Starter Kit with it.  Refer to [this
page](https://github.com/libusb/libusb/wiki/Windows#How_to_use_libusb_on_Windows)
or [this
page](https://github.com/foss-for-synopsys-dwc-arc-processors/arc_gnu_eclipse/wiki/How-to-Use-OpenOCD-on-Windows)
for details. In a nutshell, download [Zadig](http://zadig.akeo.ie/), run it and
use it to install WinUSB driver for FTDI device. If FTDI device is not shown by
Zadig, then tick "List all devices" in "Options". Note that antivirus might
complain about driver files created by Zadig. After installing the driver
everything is the same as on Linux: run OpenOCD, connect to it with GDB, etc.


## How to use OpenOCD on macOS

> Connection host is a host that is connected to the debug target via USB cable
> and runs OpenOCD. Debug host is a hsot that runs GDB, which connects to the
> OpenOCD with TCP connection. Ehily typically it is the same host, they
> actually can be different host and it is important to distinguish them.

Connect debug target to the connection host. AXS10x products and EM Starter Kit
have built-in debug cable, the don't require a separate Digilent HS cable. HS
is required only for other debug targets like ML-509 board, etc.

Run ioreg application to ensure that FTDI device is there:

    $ ioreg -p IOUSB -l -w 0

There should be output like this:
    Digilent USB Device@14100000  <class AppleUSBDevice, id 0x1000015f7, registered, matched, active, busy 0 (5 ms), retain 21>
    {
      ...
    }

Note that exact output could differ from host to host.

If you're using a USB adapter and have a driver kext matched to it,
you will need to unload it prior to running OpenOCD. E.g. with Apple
driver (OS X 10.9 or later) for FTDI run:
  sudo kextunload -b com.apple.driver.AppleUSBFTDI
for FTDI vendor driver use:
  sudo kextunload FTDIUSBSerialDriver.kext

Run OpenOCD:

See for tag Run OpenOCD at How to use OpenOCD on Linux.


## Advanced debug commands

With the GDB "monitor" command, you have "direct" access to the core without
any interference from GDB anymore! With other words, GDB has no notion of
changes in core state when using the so called  monitor commands (but it is
very powerful). In GDB, connect to the OpenOCD target and type following
command to get a list of available monitor commands:

    (gdb) monitor help

To get a list of some ARC-specific commands, run:

    (gdb) monitor help arc

Those are actually internal OpenOCD commands which are also available in
configuration scripts and can be passed in OpeOCD command line with `-c ...`
options. Those command allow to enabled some extra features of OpenOCD
(disabled by default for one or another reason) or perform some low-level
actions bypassing GDB or even OpenOCD. For example it is possible to write/read
core and aux registers. However some command for register access will be
removed in future, when ARC OpenOCD will fully support flexible register
configurations.
