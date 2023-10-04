# Building U-Boot for HS Development Kit

!!! warning

    This article is under construction!

!!! info

    * Follow [How to Get The Toolchain](../../toolchain/index.md#how-to-get-the-toolchain)
      guide to get the latest ARC GNU toolchain. Linux toolchain for ARC HS3x/4x (with uClibc or glibc)
      is required.
    * Refer to the original [U-Boot README for HSDK](https://github.com/Screenly/u-boot/blob/master/board/synopsys/hsdk/README)
      for details.

## Prerequisites

First of all U-Boot sources are fetch using next commands:

```shell
wget https://ftp.denx.de/pub/u-boot/u-boot-2023.07.tar.bz2
tar -xf u-boot-2023.07.tar.bz2
cd u-boot-2023.07
```

## Building and Writing U-Boot to SPI FLash

Build the U-Boot image for HS Development Kit:

```shell
export CROSS_COMPILE=arc-linux-

# Configure for HS Development Kit
make hsdk_defconfig

# Configure for HS Development Kit 4xD
make hsdk_4xd_defconfig

make bsp-generate
```

Run the U-Boot image on the board using `mdb`:

```shell
mdb -digilent -prop=dig_speed=10000000 -cl -run u-boot
```

Once U-Boot boots into command prompt just issue the following command:

```shell
run upgrade
```

## Building and Loading U-Boot using JTAG

Build the U-Boot image:

```shell
export CROSS_COMPILE=arc-linux-
make hsdk_defconfig
make mdbtrick
```

Ashling Opella-XD

```shell
mdb -DLL=opxdarc.so -prop=jtag_frequency=12MHz -prop=jtag_optimise=1 -nooptions -OK -memxfersize=0x8000 u-boot
```

Digilent

```shell
mdb -digilent -prop=dig_speed=10000000 u-boot
```

## Preparing U-Boot for automatic load of Linux kernel

Once U-Boot is loaded on the HSDK board it could be used for loading Linux kernel image from different media manually and automatically.

On execution of U-Boot you'll see this in serial console:

```text
U-Boot 2019.4 (May 5 2019 - 18:43:19 +0300)

CPU:   ARC HS38 v2.1c
Model: snps,hsdk
DRAM:  1 GiB
Relocation Offset is: 3ef7f000
MMC:   Synopsys Mobile storage: 0
Loading Environment from FAT... OK
In:    serial0@f0005000
Out:   serial0@f0005000
Err:   serial0@f0005000
Clock values are saved to environment
Net:   
Warning: ethernet@f0008000 (eth0) using random MAC address - 96:f3:fa:6c:19:d9
eth0: ethernet@f0008000
Hit any key to stop autoboot:  0 
hsdk#
```

Press any key to stop count-down and make sure U-Boot doesn't go to execute
automatic boot sequence.

U-Boot is controlled with its specific set of commands. List of available
commands could be obtained with `?` or `help` command like that:

```shell
hsdk# ?
?       - alias for 'help'
base    - print or set address offset
bdinfo  - print Board Info structure
boot    - boot default, i.e., run 'bootcmd'
bootd   - boot default, i.e., run 'bootcmd'
bootelf - Boot from an ELF image in memory
bootm   - boot application image from memory
bootp   - boot image via network using BOOTP/TFTP protocol
bootvx  - Boot vxWorks from an ELF image
cmp     - memory compare
coninfo - print console devices and information
cp      - memory copy
crc32   - checksum calculation
dhcp    - boot image via network using DHCP/TFTP protocol
dm      - Driver model low level access
echo    - echo args to console
editenv - edit environment variable
eeprom  - EEPROM sub-system
env     - environment handling commands
fatinfo - print information about filesystem
fatload - load binary file from a dos filesystem
fatls   - list files in a directory (default /)
fatsize - determine a file's size
fdt     - flattened device tree utility commands
go      - start application at address 'addr'
help    - print command description/usage
iminfo  - print header information for application image
imxtract- extract a part of a multi-image
itest   - return true/false on integer compare
loadb   - load binary file over serial line (kermit mode)
loads   - load S-Record file over serial line
loadx   - load binary file over serial line (xmodem mode)
loady   - load binary file over serial line (ymodem mode)
loop    - infinite loop on address range
md      - memory display
mm      - memory modify (auto-incrementing address)
mmc     - MMC sub system
mmcinfo - display MMC info
mw      - memory write (fill)
nand    - NAND sub-system
nboot   - boot from NAND device
nfs     - boot image via network using NFS protocol
nm      - memory modify (constant address)
ping    - send ICMP ECHO_REQUEST to network host
printenv- print environment variables
reset   - Perform RESET of the CPU
run     - run commands in an environment variable
saveenv - save environment variables to persistent storage
setenv  - set environment variables
sleep   - delay execution for some time
source  - run script from memory
tftpboot- boot image via network using TFTP protocol
usb     - USB sub-system
usbboot - boot from USB device
version - print monitor, compiler and linker version
```

It's also possible to create scripts - sequences of commands to be executed one
by one. And a script with special name `bootcmd` is automatically executed
after boot delay gets expired.

And so we may tune that script for our purposes.

* To load Linux image from SD-card: `setenv bootcmd fatload mmc 0\; bootm`
* To load Linux image from TFTP server: `setenv bootcmd dhcp\; bootm`

When all preparations are done it's good to save modifications of U-Boot
environment with `saveenv` command.

And with `boot` command or with restart of U-Boot (whether with reset button if
U-Boot is pre-programmed in SPI flash or with reload of U-Boot elf with MDB)
you'll be able to load and start Linux kernel.
