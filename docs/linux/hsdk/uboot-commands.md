# U-Boot Command Reference for HSDK/HSDK-4xD

The U-Boot bootloader is a powerful and flexible tool for loading an executable
from different sources and running it on the target platform.

The U-Boot implementation for the ARC HSDK/HSDK-4xD was further extended with additional
functionality that allows users to better manage the broad configurability of
the ARC HSDK/HSDK-4xD.

This command reference provides description and basic usage information for
newly introduced function in U-Boot for ARC HSDK/HSDK-4xD.

## Clock Tree Manipulations

The HSDK/HSDK-4xD board internally uses a number of PLLs that clock the main components
of the ARC core as well as all data buses and peripherals. `hsdk_clock` commands
are introduced to set and display various clock values.

### `hsdk_clock set`

Set clock from `axi_freq`, `cpu_freq` and `tun_freq` environment variables (all
values are in MHz):

```
hsdk_clock set
```

Set clock from `axi_freq`, `cpu_freq` and `tun_freq` command line arguments (all
values are in MHz):

```
hsdk_clock set tun_freq 125 axi_freq 600 cpu_freq 400
```

### `hsdk_clock get`

Save clock frequencies to `axi_freq`, `cpu_freq` and `tun_freq` environment
variables:

```
hsdk_clock get
```

### `hsdk_clock print`

Show CPU, AXI, DDR and TUNNEL current clock frequencies:

```
hsdk_clock print
```

Sample output of `hsdk_clock print` command:

```
hsdk# hsdk_clock print
HSDK: clock 'cpu-clk' rate 500 MHz
HSDK: clock 'tun-clk' rate 50 MHz
HSDK: clock 'axi-clk' rate 400 MHz
HSDK: clock 'ddr-clk' rate 334 MHz
```

### `hsdk_clock print_all`

Show all currently used clock frequencies:

```
hsdk_clock print_all
```

Typical output of `hsdk_clock print_all` command for HSDK:

```
hsdk# hsdk_clock print_all
HSDK: clock 'cpu-pll' rate 500 MHz
HSDK: clock 'cpu-clk' rate 500 MHz

HSDK: clock 'sys-pll' rate 400 MHz
HSDK: clock 'apb-clk' rate 200 MHz
HSDK: clock 'axi-clk' rate 400 MHz
HSDK: clock 'eth-clk' rate 400 MHz
HSDK: clock 'usb-clk' rate 400 MHz
HSDK: clock 'sdio-clk' rate 400 MHz
HSDK: clock 'gfx-core-clk' rate 400 MHz
HSDK: clock 'gfx-dma-clk' rate 400 MHz
HSDK: clock 'gfx-cfg-clk' rate 200 MHz
HSDK: clock 'dmac-core-clk' rate 400 MHz
HSDK: clock 'dmac-cfg-clk' rate 200 MHz
HSDK: clock 'sdio-ref-clk' rate 100 MHz
HSDK: clock 'spi-clk' rate 34 MHz
HSDK: clock 'i2c-clk' rate 200 MHz
HSDK: clock 'uart-clk' rate 34 MHz

HSDK: clock 'ddr-clk' rate 334 MHz

HSDK: clock 'tun-pll' rate 150 MHz
HSDK: clock 'tun-clk' rate 50 MHz
HSDK: clock 'rom-clk' rate 150 MHz
HSDK: clock 'pwm-clk' rate 75 MHz
```

Typical output of `hsdk_clock print_all` command for HSDK-4xD:

```
hsdk-4xd# hsdk_clock print_all
HSDK: clock 'cpu-pll' rate 500 MHz
HSDK: clock 'cpu-clk' rate 500 MHz

HSDK: clock 'sys-pll' rate 800 MHz
HSDK: clock 'apb-clk' rate 200 MHz
HSDK: clock 'axi-clk' rate 400 MHz
HSDK: clock 'eth-clk' rate 400 MHz
HSDK: clock 'usb-clk' rate 400 MHz
HSDK: clock 'sdio-clk' rate 400 MHz
HSDK: clock 'hdmi-sys-clk' rate 400 MHz
HSDK: clock 'gfx-core-clk' rate 400 MHz
HSDK: clock 'dmac-core-clk' rate 400 MHz
HSDK: clock 'dmac-cfg-clk' rate 200 MHz
HSDK: clock 'sdio-ref-clk' rate 100 MHz
HSDK: clock 'spi-clk' rate 34 MHz
HSDK: clock 'i2c-clk' rate 200 MHz
HSDK: clock 'uart-clk' rate 34 MHz

HSDK: clock 'ddr-clk' rate 334 MHz

HSDK: clock 'hdmi-pll' rate 27 MHz
HSDK: clock 'hdmi-clk' rate 27 MHz

HSDK: clock 'tun-pll' rate 600 MHz
HSDK: clock 'tun-clk' rate 50 MHz
HSDK: clock 'rom-clk' rate 150 MHz
HSDK: clock 'pwm-clk' rate 75 MHz
HSDK: clock 'timer-clk' rate 50 MHz
```

## Booting Linux from uImage

HSDK/HSDK-4xD U-Boot contains a special implementation of the `bootm` command.
HSDK/HSDK-4xD `bootm` command launches Linux kernel only on selected CPUs,
based on the value of the `core_mask` environment variable. When an external
Linux device tree blob (for example `hsdk.dtb`) is provided, bootm will also
patch the status property of each CPU node according to the value of the
`core_mask` environment variable.

Below is an example launching Linux on 3 out of 4 cores of HSDK. Linux image (uImage)
and Linux device tree blob (`hsdk.dtb`) are read from the first FAT partition
of SD card. `core_mask` is implicitly set to `0x7` by the `hsdk_hs38x3` script
command:

```
fatload mmc 0:1 0x82000000 uImage
fatload mmc 0:1 0x81000000 hsdk.dtb
run hsdk_hs38x3;
bootm 0x82000000 - 0x81000000
```

The same example for HSDK-4xD but with `hsdk_hs48x3` script command:

```
fatload mmc 0:1 0x82000000 uImage
fatload mmc 0:1 0x81000000 hsdk.dtb
run hsdk_hs48x3;
bootm 0x82000000 - 0x81000000
```

## Launching Baremetal Application

HSDK/HSDK-4xD U-Boot contains several configuration patterns which can be used to
setup HSDK/HSDK-4xD hardware in. Each configuration pattern contains information
about instruction and data caches state (enabled/disabled), `iccm` and
`dccm` mappings, `non_volatile_limit` value and CPU mask (`core_mask`
variable) value.

As of today HSDK/HSDK-4xD U-Boot contains next configuration patterns:

| HSDK            | HSDK-4xD                               |
|-----------------|----------------------------------------|
| `hsdk_hs34`     | `hsdk_hs45d`                           |
| `hsdk_hs36`     | `hsdk_hs47d`                           |
| `hsdk_hs36_ccm` | `hsdk_hs47d_ccm`                       |
| `hsdk_hs38`     | `hsdk_hs47dx2`                         |
| `hsdk_hs38_ccm` | `hsdk_hs47dx3`                         |
| `hsdk_hs38x2`   | `hsdk_hs47dx4`                         |
| `hsdk_hs38x3`   | `hsdk_hs48`                            |
| `hsdk_hs38x4`   | `hsdk_hs48_ccm`                        |
|                 | `hsdk_hs48x2` (same as `hsdk_hs47dx2`) |
|                 | `hsdk_hs48x3` (same as `hsdk_hs47dx3`) |
|                 | `hsdk_hs48x4` (same as `hsdk_hs47dx4`) |

To choose a configuration use `run` command with configuration pattern name
(`hsdk_hs38x2` configuration is used here but the same procedures are
applicable for HSDK-4xD configurations):

```
run hsdk_hs38x2
```

To setup the HSDK/HSDK-4xD hardware use `hsdk_init` command:

```
hsdk# hsdk_init 
CPU start mask is 0x3
```

To launch a baremetal application on HSDK/HSDK-4xD you need to setup entry points
for all used CPUs, load application binary to DDR, configure hardware
with `hsdk_init` command and run application with `hsdk_go` command:

```
fatload mmc 0:1 0x90000000 app.bin
run hsdk_hs38x2;
setenv core_entry_0 0x90000000;
setenv core_entry_1 0x90000000;
hsdk_init
hsdk_go
```

`hsdk_go` is command to jump each CPU chosen by CPU mask (`core_mask` variable) to
its entry point (`core_entry_X` variable). All CPUs which aren't chosen by CPU
mask will be halted. `hsdk_go` can be used with halt option to halt all CPUs
before jump to entry points:

```
hsdk_go halt
```

CPUs can be start manually with debugger.

## Running a Demo Application

You can test `hsdk_init` and `hsdk_go` commands with simple demo application:

```gas
.global _start
_start:
	# Invalidate i$, d$
	mov	r5, 1
	sr	r5, [0x10]
	mov	r5, 1
	sr	r5, [0x47]

	lr	r2, [4]
	mov	r1, 0x0000FF00
	and	r3, r2, r1
	asr_s	r3, r3, 1	# r3 == (aux(ARC_IDENTITY) & 0x0000FF00) << 7

	mov	r0, 0xD0000000
	add_s	r0, r0, r3
	mov	r1, 0x12345678
	# Write test pattern
	st	r1, [r0]
	st	r2, [r0, 0x4]

	# Flush entire D$
	mov     r5, 1
	sr      r5, [0x4B]
	# Wait for flush operation to complete
flush_wait:
	lr      r5, [0x48]
	bbit1   r5, 0x8, flush_wait	# Check status of the data-cache flush

	# Write test pattern bypassing caches.
	# **NEVER** use such code! It blows your whole leg off.
	# We use it only to make multicore test app as small as possible
	st.di	r1, [r0]
	st.di	r2, [r0, 0x4]

	nop
	nop
	nop

	flag 1
```

You can use [a prebuilt demo application binary](./app.bin) or build it yourself with next commands:

* With GNU toolchain:

    ```
    arc-linux-gcc -mcpu=hs -nostdlib -e _start -Wl,-Ttext=0x90000000 -o demo.elf demo.S
    arc-linux-objcopy -O binary demo.elf demo.bin
    ```

* With MetaWare toolchain:

    ```
    ccac -arcv2hs -Bbase=0x90000000 -g -Hnoivt -Hnolib -Hnocrt demo.S -o demo.elf
    elf2bin demo.elf demo.bin
    ```

Demo application has next functionality:

```
write 0x12345678 to (0xD0000000 + CPU_ID * 128)
write ARC_IDENTITY to (0xD000004 + CPU_ID * 128)
flush d$
halt cpu
```

Before test launching please deploy the most recent `sdcard.img` to your micro
SD-card that will be used with HSDK/HSDK-4xD and copy demo application binary to first
SD-card partition. Check ARC HSDK/HSDK-4xD Buildroot release page page for instructions
on how to deploy `sdcard.img` to micro SD-card.

There is an example of launching demo application on `hsdk_hs38x4` configuration:

```
fatload mmc 0:1 0x90000000 demo.bin
run hsdk_hs38x4;
setenv core_entry_0 0x90000000;
setenv core_entry_1 0x90000000;
setenv core_entry_2 0x90000000;
setenv core_entry_3 0x90000000;
hsdk_init
hsdk_go
```

To check demo app execution result simply read `0xD0000XXX` addresses with `MDB` or 
restart U-Boot (by pressing `Reset` button) and read memory with next U-Boot command:

```
md 0xd0000000 64
```

Check `0xd0000000`, `0xd0000080`, `0xd0000100` and `0xd0000180` addresses in MDB `Memory`
palette or in `md u-boot` command output. You will see next pattern:

```
d0000000: 12345678 00000054 XXXXXXXX XXXXXXXX
d0000080: 12345678 00000154 XXXXXXXX XXXXXXXX
d0000100: 12345678 00000254 XXXXXXXX XXXXXXXX
d0000180: 12345678 00000354 XXXXXXXX XXXXXXXX
```

## U-Boot Image Update in on-board SPI-flash

It is possible to update the U-Boot image that is automatically started
by boot-ROM with a newer image using the upgrade command provided in the
default U-Boot environment:

```
hsdk# run upgrade
reading u-boot.head
355204 bytes read in 25 ms (13.5 MiB/s)
SF: Detected sst26wf016 with page size 256 Bytes, erase size 4 KiB, total 2 MiB
SF: 393216 bytes @ 0x0 Erased: OK
device 0 offset 0x0, size 0x60000
SF: 393216 bytes @ 0x0 Written: OK
u-boot update: OK
```

## U-Boot Image Recovery in on-board SPI-flash

If U-boot image in on-board SPI-flash is damaged it is possible to
recover it by sideloading new U-Boot executable in DDR of HSDK/HSDK-4xD via
JTAG with subsequent writing of binary image into SPI-flash.

But before sideloading U-Boot executable to HSDK/HSDK-4xD's DDR please deploy
the most recent `sdcard.img` to your micro SD-card that will be used with HSDK/HSDK-4xD.
Check [Building and Running ARC Linux for HS Development Kit](./build.md) page for
instructions on how to deploy `sdcard.img` to micro SD-card.

Once micro SD-card is prepared and inserted into HSDK/HSDK-4xD make sure DIP-switches
in the corner of the board are in their default positions:
`BIM` in `1:on`, `2:on` state while both `BMC` and `BCS` should be in `1:on`,
`2:on` state.

Now reset the HSDK/HSDK-4xD board by pressing `Reset` button and load
U-Boot Elf using `MDB`:

```
mdb -digilent -cl -run u-boot
```

Once U-Boot on HSDK/HSDK-4xD boots into command prompt just issue the following command:

```
run upgrade
```

## Show Help Information About Generic U-Boot Commands

The help command (short: `h` or `?`) prints online help. Without any arguments,
it prints a list of all U-Boot commands that are available in your
configuration of U-Boot. You can get detailed information for a specific
command by typing its name as argument to the help command:

```
hsdk# help sleep
sleep - delay execution for some time

Usage:
sleep N
    - delay execution for N seconds (N is _decimal_ and can be
      fractional)
```

## Known Limitations

Every application being loaded and executed by U-Boot must invalidate all caches
(including L1 instruction and data caches as well as SLC) as the first thing on
start. This is required to make sure no stale entries were left from the previously
executed U-Boot. Even though U-Boot flushes and invalidates all caches before
passing control to another application there's still a possibility a couple of
cache lines might contain instructions and data used right after cache flush and
invalidation.
