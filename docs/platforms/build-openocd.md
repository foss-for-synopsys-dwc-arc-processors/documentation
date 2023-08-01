# Building and Installing OpenOCD

## Important Notes

1. This document uses term "Digilent HS" to refer to both HS1 and HS2 cables.
   Statements, which are applicable only to the selected cable, mention it explicitly.
   Digilent HS3 hasn't been tested with OpenOCD for ARC.
2. There is NO Flash support in ARC OpenOCD.

## Install WinUSB on Windows

## Building OpenOCD for Linux

Install prerequisites for Ubuntu:

```bash
sudo apt-get install libtool git-core build-essential autoconf \
                     automake texinfo libusb-1.0-0 libusb-1.0-0-dev \
                     pkg-config
```

Install prerequisites for RHEL/CentOS 7:

```bash
sudo yum install libtool gcc autoconf automake texinfo libusb1 libusb1-devel \
                 git make which
```

Download OpenOCD sources and checkout the latest release:

    git clone -b arc-2021.09 https://github.com/foss-for-synopsys-dwc-arc-processors/openocd
    cd openocd

Configure OpenOCD (use your own `--prefix` path):

    ./bootstrap
    ./configure --enable-ftdi --disable-werror \
                --disable-doxygen-html --prefix=/tools/openocd

Also, you can pass `--enable-verbose` and `--enable-verbose-jtag-io` options for development activities.

Build and install:

    make
    make install

Configure your environment (use your own installation path):

    export PATH=/tools/openocd/bin:$PATH

Finally you need to configure udev rules in such way that OpenOCD would be able
to claim your JTAG debug cable. In common case for ARC this is an FTDI-based
device. If you already have libftdi package installed on your system, then
required rules are already provided to udev. Otherwise create file
`/etc/udev/rules.d/99-ftdi.rules` with the following contents:

    # allow users to claim the device
    # Digilent HS1 and similiar products
    SUBSYSTEM=="usb", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6010", MODE="0664", GROUP="plugdev"
    # Digilent HS2
    SUBSYSTEM=="usb", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6014", MODE="0664", GROUP="plugdev"

You also can use file `contrib/99-openocd.udev` supplied with OpenOCD sources,
however this file doesn't work with Digilent HS2, though on the other hand it
mentions many other FTDI-based devices.

Then either reboot your system or reload udev configuration and reconnect debug
cable to the host computer:

    $ sudo udevadm control --reload-rules
    # Disconnect JTAG cable from host, then connect again.

## Building OpenOCD for Windows

It is possible to use OpenOCD on Windows with FTDI-based debug cables using a
`ftdi` interface and libusb driver (further down called ftdi/libusb). Note,
however that this requires replacing the original FTDI proprietary drivers with
open source ones. This will render Digilent cable unusable by Digilent tools,
like Adept.

Install the same prerequisites like for Linux build (except for libusb-dev) and
MinGW on your system:

    $ sudo apt-get install libtool git-core build-essential autoconf automake
    texinfo pkg-config

Install MinGW cross-compiler to your system:

    $ sudo apt-get install gcc-mingw-w64

Download [libusb sources](http://libusb.info) sources. Configure and build them
with MinGW compiler. It is recommended to build only static libusb, so that
OpenOCD will not need this library's dll file to be copied around:

    $ tar xaf libusb-1.0.20.tar.bz2
    $ cd libusb-1.0.20
    $ ./configure --host=i686-w64-mingw32 --build=x86_64-linux-gnu \
      --prefix=</libusbx/install/path> --disable-shared --enable-static
    $ make
    $ make install

Download OpenOCD sources:

    $ git clone https://github.com/foss-for-synopsys-dwc-arc-processors/openocd
    $ cd openocd
    $ ./bootstrap

Configure OpenOCD. Consult `configure --help` and generic OpenOCD documentation
for details. This command line is recommended for ARC with libusb/ftdi:

    $ PKG_CONFIG_PATH=</libusb/install/path>/lib/pkgconfig ./configure \
      --enable-ftdi --host=i686-w64-mingw32 --build=x86_64-linux-gnu \
      --disable-werror --prefix=<openocd/install/path>

Note that it is required to set PKG_CONFIG_PATH, otherwise configure script
will detect host libusb installation, instead of the one cross-compiled for
Windows.

Build and install:

    $ make
    $ make install

If your application uses libusb and is being linked dynamically (this is by
default), copy </libusbx/install/path>/bin/libusb-1.0.dll to the OpenOCD bin
directory. Copy OpenOCD installation to Window host.

## Find your way in the source code

Automake makefile entry starts from: src/target/Makefile.am

In src/target is the ARC specific code base:

	Top and bottom interfaces into OpenOCD:

	arc.c				main hook into OpenOCD framework (target function body)
	arc.h				main include (gets everywhere included)
	arc_jtag.c + .h		ARC jtag interface into OpenOCD

	Supporting functions/modules as used by above interface into OpenOCD

	arc32.c + .h		generic ARC architecture functions
	arc_core.c + .h		ARC core internal specifics
	arc_dbg.c + .h		ARC debugger functions
	arc_mem.c + .h		ARC memory functions
	arc_mntr.c + .h		GDB monitor functions
	arc_ocd.c + .h		ARC OCD initialization
	arc_regs.c + .h		ARC register access
	arc_trgt.c + .h		target/board system functions


## Digilent driver installation instructions

OpenOCD doesn't use Digilent drivers to communicate with Digilent debug cables,
instead it uses it's own implementation of FTDI MPSSE protocol, which us
compatible with _virtually_ any other FTDI x232 cable. However to use some
features of Digilent cables you might need to install their drivers and
utilities. Following are instructions on how to do that.

Download appropriate version of runtime and utilities from Digilent site:
http://www.digilentinc.com/Products/Detail.cfm?Prod=ADEPT2 .

Untar and install products:

    $ tar xzfv digilent.adept.runtime_2.10.2-i686.tar.gz
    $ cd digilent.adept.runtime_2.10.2-i686
    $ sudo ./install.sh
    $ cd ftdi.drivers_1.0.4-i686
    $ sudo ./install.sh
    $ cd ../..

    $ tar xvzf digilent.adept.utilities_2.1.1-i686.tar.gz
    $ cd digilent.adept.utilities_2.1.1-i686
    $ sudo ./install.sh
    $ cd ..

## How to program a bit-file into FPGA with the Digilent HS cable

Below is an example given of how to program a Xilinx ML509 FPGA developers
board. NOTE: make sure the Digilent HS cable is connected to the right
JTAG connector on the board (programming the FPGA and not the memory).
So, it should be connected to PC4 JTAG and not to J51 BDM.
Further more, Device 4: XC5VLX110T is the FPGA to program, device 4 in the
JTAG scan chain.

    > djtgcfg enum
    Found 1 device(s)

    Device: JtagHs2
        Product Name:   Digilent JTAG-HS2
        User Name:      JtagHs2
        Serial Number:  210249810909

    > djtgcfg -d JtagHs2 init
    Initializing scan chain...
    Found Device ID: a2ad6093
    Found Device ID: 0a001093
    Found Device ID: 59608093
    Found Device ID: f5059093
    Found Device ID: f5059093

    Found 5 device(s):
        Device 0: XCF32P
        Device 1: XCF32P
        Device 2: XC95144XL
        Device 3: XCCACE
        Device 4: XC5VLX110T

    > djtgcfg -d JtagHs2 prog -i 4 -f <fpga bit file to progam>.bit
    Programming device. Do not touch your board. This may take a few minutes...
    Programming succeeded.

## Running internal testsuite

There is a set of internal test for ARC and OpenOCD (consisting of just a
single test at the moment of this writing, though...). This testsuite aim is to
catch some issues with OpenOCD, JTAG or hardware. To run test suite: source
`tcl/test/arc.cfg` then run arc_test_run_all procedure, or run tests
individually.
