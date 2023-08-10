# Building and Running ARC Linux for AXS103 Software Development Platform

## Preparing the Board

The AXS103 development system consists of an ARC SDP mainboard and a new AXC003
card with an FPGA containing the ARC HS38x2 (dual core). Please refer to
[board's documentation](https://github.com/foss-for-synopsys-dwc-arc-processors/ARC-Development-Systems-Forum/wiki/ARC-Development-Systems-Forum-Wiki-Home#arc-axs103-software-development-platform-1)
for detailed information about how to setup the board for initial operation.

## Connecting to a Serial Port Terminal

1. Connect USB-keyboard to the USB-port of the board
2. Connect USB-cable from your host PC to the DataPort (used for serial
   communication and for built-in Digilent JTAG)

Debug console will be instantiated on so-called "data-port".
As a result, we may interact with running Linux distribution by means of serial console.

To use serial console we must setup serial-port terminal on host PC/laptop. In
case of Windows we may use HyperTerminal or Putty, in case of Linux host we may
use `minicom`, `gtkterm` or `cutecom`. Terminal client must be set to baud-rate
115200, 8 bits.

## Building the Linux Image

Get Buildroot from the upstream repository:

```shell
wget http://buildroot.uclibc.org/downloads/buildroot-2017.02.tar.gz
tar -xvf buildroot-2017.02.tar.gz
cd buildroot-2017.02
```

Download the following patches for this version of Buildroot that are not yet accepted upstream.

* [0001-toolchain-Disable-PIE-for-ARC.patch](patches/0001-toolchain-Disable-PIE-for-ARC.patch)
* [0002-axs103-v1.1-Prepare-for-release.patch](patches/0002-axs103-v1.1-Prepare-for-release.patch)

Apply those patches on top of extracted source tree of Buildroot:

```shell
patch -p1 < 0001-toolchain-Disable-PIE-for-ARC.patch
patch -p1 < 0002-axs103-v1.1-Prepare-for-release.patch
```

Now configure Buildroot to use default configuration for AXS103 and execute building procedure:

```shell
make snps_axs103_defconfig
make
```

Building takes some time but usually less than an hour on modern machines.
Note that build will produce following files in `output/images/` folder:

* `rootfs.cpio`, `rootfs.tar` - these 2 files contain minimalistic root filesystem,
  for example `rootfs.cpio` could be re-used when manually building Linux kernel for ARC boards.
* `u-boot.bin` - this is a binary image of U-Boot bootloader, it is meant to be
  programmed in AXS' SPI flash and then autostart on power-on.
* `uImage` - this is Linux kernel prepared for loading by U-Boot bootloader.
  When `u-boot.bin` is programmed in AXS' SPI flash and autostarts on power-on
  it will attempt to find uImage on the first partition of the SD-card. But if
  one wants to load Linux kernel directly via JTAG then `vmlinux` should be
  built, for that run `make menuconfig`, go to `Kernel -> Kernel binary format`
  and  select `vmlinux` (BR2_LINUX_KERNEL_VMLINUX).

## Running the Linux Image Using MetaWare Debugger via JTAG

After Buildroot has successfully created the image we need to load this Linux
image onto the target and run it. And that's exactly the case when we want to
have `vmlinux` created by Buildroot instead of `uImage`, so follow instructions
above and enable `BR2_LINUX_KERNEL_VMLINUX`.

Make sure JTAG-probe is attached to the board. In case of Ashling Opella-XD
the following command-line should be used for MetaWare Debugger:

```shell
mdb -pset=1 -psetname=core0 -DLL=opxdarc.so -prop=jtag_frequency=12MHz -prop=jtag_optimise=1 -memxfersize=0x8000 output/images/vmlinux
mdb -pset=2 -psetname=core1 -DLL=opxdarc.so -prop=jtag_frequency=12MHz -prop=jtag_optimise=1 -prop=download=2 output/images/vmlinux
mdb -multifiles=core0,core1 -run -cl
```

In case of Digilent HS1/HS2 the following command-line should be used:

```shell
mdb -pset=1 -psetname=core0 -digilent -prop=dig_speed=10000000 output/images/vmlinux
mdb -pset=2 -psetname=core1 -digilent -prop=download=2 output/images/vmlinux
mdb -multifiles=core0,core1 -run -cl
```

It's possible to use OpenOCD and Digilent HS1/HS2 probe for loading and debugging
the Linux kernel on HSDK. You can find detailed instructions in
[Using OpenOCD](../../platforms/use-openocd.md) guide.

At this point the following log messages should appear in the console:

```text
Linux version 4.10.8 (abrodkin@ru20arcgnu1) (gcc version 6.2.1 20160824 (Buildroot 2017.02-00002-g1ad32dd67-dirty) ) #2 SMP PREEMPT Wed Apr 5 21:24:07 MSK 2017
Memory @ 80000000 [512M] 
OF: fdt:Machine model: snps,axs103-smp
earlycon: uart8250 at MMIO32 0xe0022000 (options '115200n8')
bootconsole [uart8250] enabled
Freq is 100MHz
AXS: AXC003 CPU Card FPGA Date: 7-3-2017
AXS: MainBoard v3 FPGA Date: 7-3-2017
archs-intc      : 2 priority levels (default 1)

IDENTITY        : ARCVER [0x53] ARCNUM [0x0] CHIPID [ 0x0]
processor [0]   : ARC HS38 R3.0 (ARCv2 ISA) 
Timers          : Timer0 Timer1 GFRC [SMP 64-bit] 
ISA Extn        : atomic ll64 unalign (not used)
                : mpy[opt 9] div_rem norm barrel-shift swap minmax swape
BPU             : full match, cache:512, Predict Table:8192
MMU [v4]        : 8k PAGE, 2M Super Page (not used) JTLB 512 (128x4), uDTLB 8, uITLB 4, PAE40 (not used) 
I-Cache         : 64K, 4way/set, 64B Line, VIPT aliasing
D-Cache         : 64K, 2way/set, 64B Line, PIPT
SLC             : 512K, 64B Line
Peripherals     : 0xe0000000, IO-Coherency 
Vector Table    : 0x80000000
FPU             : SP DP 
DEBUG           : ActionPoint smaRT RTT 
OS ABI [v4]     : 64-bit data any register aligned
Extn [SMP]      : ARConnect (v2): 2 cores with IPI IDU DEBUG GFRC
CONFIG_ARC_FPU_SAVE_RESTORE needed for working apps
Reserved memory: created DMA memory pool at 0xbe000000, size 32 MiB
OF: reserved mem: initialized node frame_buffer@be000000, compatible id shared-dma-pool
percpu: Embedded 6 pages/cpu @9fd64000 s16384 r8192 d24576 u49152
Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 65248
Kernel command line: earlycon=uart8250,mmio32,0xe0022000,115200n8 console=tty0 console=ttyS3,115200n8 print-fatal-signals=1 consoleblank=0 video=1280x720@60
PID hash table entries: 2048 (order: 0, 8192 bytes)
Dentry cache hash table entries: 65536 (order: 5, 262144 bytes)
Inode-cache hash table entries: 32768 (order: 4, 131072 bytes)
Memory: 504528K/524288K available (4558K kernel code, 150K rwdata, 888K rodata, 10112K init, 273K bss, 19760K reserved, 0K cma-reserved)
Preemptible hierarchical RCU implementation.
        Build-time adjustment of leaf fanout to 32.
NR_IRQS:128
MCIP: IDU referenced from Devicetree 2 irqs
clocksource: ARConnect GFRC: mask: 0xffffffffffffffff max_cycles: 0x171024e7e0, max_idle_ns: 440795205315 ns
Console: colour dummy device 80x25
console [tty0] enabled
Calibrating delay loop... 99.22 BogoMIPS (lpj=496128)
pid_max: default: 32768 minimum: 301
Mount-cache hash table entries: 2048 (order: 0, 8192 bytes)
Mountpoint-cache hash table entries: 2048 (order: 0, 8192 bytes)
smp: Bringing up secondary CPUs ...
Idle Task [1] 9f053880
Trying to bring up CPU1 ...
archs-intc      : 2 priority levels (default 1)

IDENTITY        : ARCVER [0x53] ARCNUM [0x1] CHIPID [ 0x0]
processor [1]   : ARC HS38 R3.0 (ARCv2 ISA) 
Timers          : Timer0 Timer1 
ISA Extn        : atomic ll64 unalign (not used)
                : mpy[opt 9] div_rem norm barrel-shift swap minmax swape
BPU             : full match, cache:512, Predict Table:8192
MMU [v4]        : 8k PAGE, 2M Super Page (not used) JTLB 512 (128x4), uDTLB 8, uITLB 4, PAE40 (not used) 
I-Cache         : 64K, 4way/set, 64B Line, VIPT aliasing
D-Cache         : 64K, 2way/set, 64B Line, PIPT
SLC             : 512K, 64B Line
Peripherals     : 0xe0000000, IO-Coherency 
Vector Table    : 0x80000000
FPU             : SP DP 
DEBUG           : ActionPoint smaRT RTT 
OS ABI [v4]     : 64-bit data any register aligned
Extn [SMP]      : ARConnect (v2): 2 cores with IPI IDU DEBUG GFRC
CONFIG_ARC_FPU_SAVE_RESTORE needed for working apps
## CPU1 LIVE ##: Executing Code...
Idle Task [2] 9f0533c0
Trying to bring up CPU2 ...
Timeout: CPU2 FAILED to comeup !!!
Idle Task [3] 9f052f00
Trying to bring up CPU3 ...
random: fast init done
Timeout: CPU3 FAILED to comeup !!!
smp: Brought up 1 node, 2 CPUs
devtmpfs: initialized
clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
futex hash table entries: 1024 (order: 3, 65536 bytes)
NET: Registered protocol family 16
irq: no irq domain found for /cpu_card/dw-apb-gpio@0x2000/gpio-controller@0 !
SCSI subsystem initialized
usbcore: registered new interface driver usbfs
usbcore: registered new interface driver hub
usbcore: registered new device driver usb
i2c_designware e001e000.i2c: Unknown Synopsys component type: 0x00000030
pps_core: LinuxPPS API ver. 1 registered
pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
PTP clock support registered
clocksource: Switched to clocksource ARConnect GFRC
NET: Registered protocol family 2
TCP established hash table entries: 4096 (order: 1, 16384 bytes)
TCP bind hash table entries: 4096 (order: 2, 32768 bytes)
TCP: Hash tables configured (established 4096 bind 4096)
UDP hash table entries: 256 (order: 0, 8192 bytes)
UDP-Lite hash table entries: 256 (order: 0, 8192 bytes)
NET: Registered protocol family 1
RPC: Registered named UNIX socket transport module.
RPC: Registered udp transport module.
RPC: Registered tcp transport module.
RPC: Registered tcp NFSv4.1 backchannel transport module.
ARC perf        : 8 counters (48 bits), 121 conditions, [overflow IRQ support]
workingset: timestamp_bits=30 max_order=16 bucket_order=0
ntfs: driver 2.1.32 [Flags: R/O].
Block layer SCSI generic (bsg) driver version 0.4 loaded (major 251)
io scheduler noop registered
io scheduler deadline registered
io scheduler cfq registered (default)
Serial: 8250/16550 driver, 4 ports, IRQ sharing disabled
f0005000.dw-apb-uart: ttyS0 at MMIO 0xf0005000 (irq = 6, base_baud = 2083312) is a 16550A
e0020000.uart: ttyS1 at MMIO 0xe0020000 (irq = 17, base_baud = 2083333) is a 16550A
e0021000.uart: ttyS2 at MMIO 0xe0021000 (irq = 18, base_baud = 2083333) is a 16550A
e0022000.uart: ttyS3 at MMIO 0xe0022000 (irq = 22, base_baud = 2083333) is a 16550A
console [ttyS3] enabled
console [ttyS3] enabled
bootconsole [uart8250] disabled
bootconsole [uart8250] disabled
loop: module loaded
libphy: Fixed MDIO Bus: probed
stmmaceth e0018000.ethernet: no reset control found
stmmac - user ID: 0x10, Synopsys ID: 0x37
stmmaceth e0018000.ethernet: Ring mode enabled
stmmaceth e0018000.ethernet: DMA HW capability register supported
stmmaceth e0018000.ethernet: Normal descriptors
stmmaceth e0018000.ethernet: RX Checksum Offload Engine supported
stmmaceth e0018000.ethernet: COE Type 2
stmmaceth e0018000.ethernet: TX Checksum insertion supported
stmmaceth e0018000.ethernet: Enable RX Mitigation via HW Watchdog Timer
libphy: stmmac: probed
stmmaceth e0018000.ethernet (unnamed net_device) (uninitialized): PHY ID 20005c7a at 1 IRQ POLL (stmmac-0:01) active
ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
ehci-platform: EHCI generic platform driver
ehci-platform e0040000.ehci: EHCI Host Controller
ehci-platform e0040000.ehci: new USB bus registered, assigned bus number 1
ehci-platform e0040000.ehci: irq 8, io mem 0xe0040000
ehci-platform e0040000.ehci: USB 2.0 started, EHCI 1.00
hub 1-0:1.0: USB hub found
hub 1-0:1.0: 1 port detected
ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
ohci-platform: OHCI generic platform driver
ohci-platform e0060000.ohci: Generic Platform OHCI controller
ohci-platform e0060000.ohci: new USB bus registered, assigned bus number 2
ohci-platform e0060000.ohci: irq 8, io mem 0xe0060000
hub 2-0:1.0: USB hub found
hub 2-0:1.0: 1 port detected
usbcore: registered new interface driver usb-storage
mousedev: PS/2 mouse device common for all mice
usbcore: registered new interface driver synaptics_usb
i2c /dev entries driver
sdhci: Secure Digital Host Controller Interface driver
sdhci: Copyright(c) Pierre Ossman
Synopsys Designware Multimedia Card Interface Driver
dw_mmc e0015000.mmc: IDMAC supports 32-bit address mode.
dw_mmc e0015000.mmc: Using internal DMA controller.
dw_mmc e0015000.mmc: Version ID is 290a
dw_mmc e0015000.mmc: DW MMC controller at irq 7,32 bit host data width,16 deep fifo
mmc_host mmc0: Bus speed (slot 0) = 50000000Hz (slot req 400000Hz, actual 396825HZ div = 63)
dw_mmc e0015000.mmc: 1 slots initialized
sdhci-pltfm: SDHCI platform and OF driver helper
usbcore: registered new interface driver usbhid
usbhid: USB HID core driver
NET: Registered protocol family 17
NET: Registered protocol family 15
ttyS3 - failed to request DMA
Freeing unused kernel memory: 10112K
This architecture does not have kernel memory protection.
usb 1-1: new high-speed USB device number 2 using ehci-platform
Starting logging: OK
Initializing random number generator... done.
Starting network: stmmaceth e0018000.ethernet eth0: device MAC address ba:24:55:7e:f8:4f
stmmaceth e0018000.ethernet eth0: fail to init PTP.
udhcpc: started, v1.26.2
udhcpc: sending discover
stmmaceth e0018000.ethernet eth0: Link is Up - 100Mbps/Full - flow control off
udhcpc: sending discover
udhcpc: sending select for 10.42.0.225
udhcpc: lease of 10.42.0.225 obtained, lease time 3600
deleting routers
adding dns 10.42.0.1
OK
ssh-keygen: generating new host keys: RSA DSA 
ECDSA ED25519 
Starting sshd: OK
random: crng init done

Welcome to the ARC Software Development Platform
axs103 login: root
```

Enter login `root` and press Enter.

Note if you see following in bootlog:
```
udhcpc (v1.23.2) started
Sending discover...
Sending discover...
Sending discover...
No lease, failing
```

And also later:

```text
stmmaceth e0018000.ethernet eth0: Link is Up - 100Mbps/Full - flow control rx/tx
```

That means Ethernet PHY became alive a bit too late. And that means it's required
to execute DHCP discovery once again from console:

```text
udhcpc
```

Now you may run whatever programs were built from your Buildroot selections.
For the default only busybox is built and installed.

```text
# ls /dev
bus                 tty1                tty43
console             tty10               tty44
cpu_dma_latency     tty11               tty45
full                tty12               tty46
gpiochip0           tty13               tty47
gpiochip1           tty14               tty48
gpiochip2           tty15               tty49
gpiochip3           tty16               tty5
gpiochip4           tty17               tty50
gpiochip5           tty18               tty51
gpiochip6           tty19               tty52
i2c-0               tty2                tty53
i2c-1               tty20               tty54
input               tty21               tty55
kmsg                tty22               tty56
log                 tty23               tty57
loop-control        tty24               tty58
loop0               tty25               tty59
loop1               tty26               tty6
loop2               tty27               tty60
loop3               tty28               tty61
loop4               tty29               tty62
loop5               tty3                tty63
loop6               tty30               tty7
loop7               tty31               tty8
mem                 tty32               tty9
memory_bandwidth    tty33               ttyS0
network_latency     tty34               ttyS1
network_throughput  tty35               ttyS2
null                tty36               ttyS3
psaux               tty37               urandom
ptmx                tty38               vcs
pts                 tty39               vcs1
random              tty4                vcsa
shm                 tty40               vcsa1
tty                 tty41               zero
tty0                tty42
```

## Running the Linux Image Using U-Boot

It's possible to use U-Boot bootloader for loading and starting Linux image on
AXS103 board. U-Boot itself could be whether pre-programmed in on-board SPI flash
and start automatically on power-on or user may load it via JTAG similarly to how
kernel image was loaded above.

### Preparing the Linux Image for Loading Using U-Boot

Linux kernel built with Buildroot as described above will not be suitable for
use with U-Boot as it requires post-processing resulting ELF-file to add U-Boot
specific header in the beginning of the image.

Fortunately Buildroot may handle it easily. Just start Buildroot's configuration
utility with `make menuconfig`, go to`Kernel -> Kernel binary format` and select
`uImage`. Save your new configuration, exit configuration utility and issue `make`
command once again. You'll get `uImage` file in `output/images/vmlinux` folder.
Now put that file in the root of FAT32-formatted SD-card or in shared folder on
TFTP server.

### Building U-Boot for AXS103 board

Anyway U-Boot for AXS103 board could be built from upstream sources starting from
`v2015.10` that way:

```shell
wget ftp://ftp.denx.de/pub/u-boot/u-boot-2017.01.tar.bz2
tar -xf u-boot-2017.01.tar.bz2
cd u-boot-2017.01
```

Download the following patches that are missing from the current U-Boot release:

```text
curl http://git.denx.de/?p=u-boot.git;a=patch;h=0b0db98be7e22f5b862b62f63db7ff6ab02a3a7f > 0001-axs103-Clean-up-smp_kick_all_cpus.patch
curl http://git.denx.de/?p=u-boot.git;a=patch;h=2a5062ca9ecc22b88af2babf812b05dd97ecde46 > 0002-axs103-Support-slave-core-kick-start-on-axs103-v1.1-.patch
```

Apply those patches on top of unpacked U-Boot source tree:

```shell
patch -p1 < 0001-axs103-Clean-up-smp_kick_all_cpus.patch
patch -p1 < 0002-axs103-Support-slave-core-kick-start-on-axs103-v1.1-.patch
```

Configure U-Boot before building:

```shell
make axs103_defconfig
```

Now depending on use-case 2 different commands should be used.

* For programming in on-board SPI flash: `make u-boot.bin`.
* For loading U-Boot with debugger via JTAG: `make mdbtrick`.

### Flashing U‐Boot

ARC AXS10x Software Development Platforms (SDP) have built-in SPI-flash storage
which may be used to host auto-started applications such as bootloaders, etc.
Detailed instructions are available in this article:  
[Flashing U‐Boot in ARC AXS10x Software Development Platforms (SDP)](./uboot.md)

### Loading U-Boot with debugger via JTAG

```shell
### Ashling Opella-XD
mdb -DLL=opxdarc.so -prop=jtag_frequency=12MHz -prop=jtag_optimise=1 -nooptions -OK -memxfersize=0x8000 u-boot

### Digilent
mdb -digilent -prop=dig_speed=10000000 u-boot
```

### Preparing U-Boot for automatic load of Linux kernel

Once U-Boot is loaded on the AXS103 board it could be used for loading Linux kernel image from different media manually and automatically.

On execution of U-Boot you'll see this in serial console:

```text
U-Boot 2017.01 (Apr 05 2017 - 15:52:58 +0300)

I2C:   ready
DRAM:  512 MiB
NAND:  512 MiB
MMC:   Synopsys Mobile storage: 0
In:    serial0@e0022000
Out:   serial0@e0022000
Err:   serial0@e0022000
Net:   
Warning: ethernet@e0018000 (eth0) using random MAC address - ba:24:55:7e:f8:4f
eth0: ethernet@e0018000
Hit any key to stop autoboot:  0
AXS#
```

Press any key to stop count-down and make sure U-Boot doesn't go to execute
automatic boot sequence. U-Boot is controlled with its specific set of commands.
List of available commands could be obtained with `?` or `help` command
like that:

```text
AXS# ?
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
by one. And a script with special name `bootcmd` is automatically executed after
boot delay gets expired.

And so we may tune that script for our purposes.

* To load Linux image from SD-card: `setenv bootcmd fatload mmc 0\; bootm`
* To load Linux image from TFTP server: `setenv bootcmd dhcp\; bootm`

Note that for loading image from TFTP server it's required:

1. To assign a valid MAC-address to AXS103 board with the following command: `setenv ethaddr 66:55:44:33:22:11`
2. Specify IP-address of TFTP server with command: `setenv serverip 10.11.12.13`

When all preparations are done it's good to save modifications of U-Boot
environment with `saveenv` command.

And with `boot` command or with restart of U-Boot (whether with reset button
if U-Boot is pre-programmed in SPI flash or with reload of U-Boot elf with MDB)
you'll be able to load and start Linux kernel.

## Video Output via HDMI

It is possible to attach a modern TV or computer monitor to ARC SDP board via
HDMI cable. Even though on ARC SDP board there's only HDMI connector it's
possible to use DVI monitor as well through DVI-HDVI converter.

By default in Linux kernels starting from 4.10 device driver for ARC PGU is
built as a kernel module and so is not loaded automatically on start. Moreover
if initramfs and/or Linux kernel is built outside of Buildroot there's a chance
to not have corresponding modules there which makes it impossible to get video
output working.

The simplest solution here is just to use Buildroot's defconfig for AXS103,
i.e. configure Buildroot with `make snps_archs38_axs103_defconfig`. Buildroot
takes care of copying kernel modules onto roofs so on target user may just say:

```shell
modprobe arcpgu.ko
modprobe adv7511.ko
```

At this point debug console will appear on the screen.

## Play Video with mplayer on HDMI Monitor

Note that mplayer application is not built by Buildroot by default for AXS103
so it is required to select corresponding package (`BR2_PACKAGE_MPLAYER`) in
Buildroot configuration utility and rebuild everything with `make` command.
Make sure that rebuilt file-system ends up on your target, i.e. use updated
`vmlinux` or `uImage`  file if rootfs is built-in Linux kernel image or deploy
file-system image on your storage (like SD-card or USB flash-drive).

Copy Big Buck Bunny video on USB flash-drive or on SD-card from here
<http://download.blender.org/peach/bigbuckbunny_movies/BigBuckBunny_320x180.mp4>

Insert you storage media into ARC SDP, mount your storage and start playback with:

```shell
mplayer -vo fbdev2 BigBuckBunny_320x180.mp4
```

Note playback will be really-really slow on axs103 due to absent hardware
acceleration of mp4 decoding and CPU frequency of only 100 MHz.
