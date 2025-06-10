# Building Linux kernel with Filesystem Using Buildroot

## Getting Buildroot

Clone the Buildroot repository:

```shell
git clone -b arc-2025.06 https://github.com/foss-for-synopsys-dwc-arc-processors/buildroot
cd buildroot
```

## Using Predefined Configurations

!!! info

    By default, `snps_archs38_haps_defconfig` uses `haps_hs_smp`
    kernel configuration file. If you are going to run an image on nSIM with
    a single core then change it to `haps_hs` through `make menuconfig`
    (`Kernel` -> `Defconfig name` -> `haps_hs`).

Buildroot uses configuration files (`defconfig`s) for configuring a root filesystem,
a toolchain, a Linux kernel, etc. Buildroot comes with a number of existing predefined
configurations which may be found in `config` directory. Use `make list-defconfigs` command
to list all available configurations:

```text
$ make list-defconfigs | grep snps
  snps_arc32_defconfig                - Build for snps_arc32
  snps_arc64_defconfig                - Build for snps_arc64
  snps_arc700_axs101_defconfig        - Build for snps_arc700_axs101
  snps_archs38_axs103_defconfig       - Build for snps_archs38_axs103
  snps_archs38_haps_defconfig         - Build for snps_archs38_haps
  snps_archs38_hsdk_defconfig         - Build for snps_archs38_hsdk
```

Here is a short description of each configuration:

* `snps_arc32_defconfig` - A configuration for ARC HS5x on HAPS, nSIM or QEMU (available only in [a separate repository](https://github.com/foss-for-synopsys-dwc-arc-processors/buildroot)).
* `snps_arc64_defconfig` - A configuration for ARC HS6x on HAPS, nSIM or QEMU (available only in [a separate repository](https://github.com/foss-for-synopsys-dwc-arc-processors/buildroot)).
* `snps_arc700_axs101_defconfig` - A configuration for ARC 700 on AXS101 development board.
* `snps_archs38_axs103_defconfig` - A configuration for ARC HS38 on AXS103 development board.
* `snps_archs38_haps_defconfig` - A configuration for ARC HS3x/4x in ZeBU, HAPS, nSIM or QEMU.
* `snps_archs38_hsdk_defconfig` - A configuration for ARC HS3x/4x on HSDK and HSDK 4xD development boards.
This configuration builds images for running Linux using U-Boot.

## Configuring Root Filesystem

You can select a predefined configuration this way:

```shell
make snps_archs38_haps_defconfig
```

You can manually tune filesystem's configuration this way:

```shell
make menuconfig
```

The final configuration file is saved in `.config` file.

## Configuring Linux Kernel

Also, you can configure kernel's options this way:

```shell
make linux-menuconfig
```

## Building Linux

Buildroot with default configurations, which are mentioned above, builds Linux images
in this sequence:

1. Download sources for a toolchain and build it.
2. Download sources for packages, build them using the toolchain and put them
in a filesystem image (`rootfs`).
3. Download sources for a Linux kernel and build it using the toolchain.
4. Link the Linux kernel against `rootfs` (`vmlinux`) or save it separately from `rootfs` (`bzImage` or `uImage`).
Whether `rootfs` is linked against  Linux kernel or not depends on a particular configuration. For example,
`snps_archs38_hsdk_defconfig` configuration doesn't link `rootfs` against Linux kernel.

You can build images using `make`:

```shell
make
```

By default, Buildroot builds everything in `output` directory. Images are placed in
`output/images` directory. For `snps_archs38_haps_defconfig` configuration you will find
a couple of filesystem images (`rootfs.cpio` and `rootfs.tar`) and `vmlinux` - a kernel
with the filesystem included:

```text
$ ls output/images
rootfs.cpio  rootfs.tar  vmlinux
```

For example, for `snps_archs38_hsdk_defconfig` configuration you will find much more
images (consider reading a guide about building Linux for HSDK for more details):

```text
$ ls -1 output/images
boot.vfat
rootfs.ext2
rootfs.ext4
sdcard.img
u-boot
u-boot.bin
uImage
uboot-env.bin
```
