# 🕥 Flashing U‐Boot in ARC AXS10x Software Development Platforms (SDP)

!!! warning

    AXS board is no longer supported. There is no guarantee that this guide will
    be applicable for the latest tools.

ARC AXS10x Software Development Platforms (SDP) have built-in SPI-flash storage
which may be used to host auto-started applications such as bootloaders, etc.

U-Boot is a good fit for such application. Very early boot SDP's pre-bootloader
application is able to find images in that SPI-flash, load them in specified
memory location and then execute it.

It's also possible to have separate images to be loaded and executed for
different CPU cores. This allows us to have U-Boot images for both ARC 770D
and ARC HS38 at once each starting when appropriate CPU tile is mounted on
the base-board.

Below are command for preparing [if needed - for ARCv2 CPUs] and flashing
U-Boot in SPI flash.

* For ARC 770D

    * Build U-Boot binary image: `make axs101_defconfig && make u-boot.bin`
    * Flash image: `axs_comm -c 0434 -a 00000000 -p 81000000 -f u-boot.bin`

* For ARC HS38

    * Build U-Boot binary image: `make axs103_defconfig && make u-boot.bin`
    * Add jump location just before the image: `printf "\x00\x00\x00\x81" | cat - u-boot.bin > u-boot-offset.bin`
    * Flash image

For ARC HS38 version 2.0 `arc_id` is `0x51`, for version 2.1 `arc_id` is `0x52`,
for version 3.0 `arc_id` is `0x53`. You can use the same u-boot image for both
versions but you need to pass different values to `axs_comm` `-c` option:

* ARC HS38 2.0: `axs_comm -c 0051 -a 00080000 -p 80FFFFFC -f u-boot-offset.bin`
* ARC HS38 2.1: `axs_comm -c 0052 -a 00100000 -p 80FFFFFC -f u-boot-offset.bin`
* ARC HS38 3.0: `axs_comm -c 0053 -a 00180000 -p 80FFFFFC -f u-boot-offset.bin`

Note for ARC HS38 3rd pin in dip-switch `SW2501` must be in `off` position
(which is closer to CPU tile). Otherwise image from SPI-flash won't be read.
