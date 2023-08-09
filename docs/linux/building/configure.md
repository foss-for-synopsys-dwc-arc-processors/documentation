# Configuring Buildroot and Linux Kernel

## Preface

This guide contains a bunch of recipes which may be useful while building Linux kernel
for Synopsys ARC processors. Consider reading [the Buildroot user manual](https://buildroot.org/downloads/manual/manual.html) for details.

## Building in a Separate Directory

It's a good practice to build Linux images out of the main source tree:

```shell
git clone https://git.busybox.net/buildroot
mkdir buildroot/build
cd buildroot/build
make -C .. O=$(pwd) snps_archs38_haps_defconfig
```

Option `-C ..` points to Buildroot source tree and `O=$(pwd)` passes an output directory's path to `make`.
Note that if you use relative paths in Buildroot configuration options, then they are relative to Buildroot's
source tree, but not to build directory.

## Selection a Big Endian Configuration

Select this option in a configuration menu:

```text
Target options -> Target Architecture -> (X) ARC (big endian)
```

## Selecting a Linux Kernel Version

By default, Buildroot selects the latest available version of the upstream Linux kernel. However, if you
want to use a development branch with the latest patches and support of ARCv3 processor families, then
consider using `arc64` branch of the development repository:

```text
Kernel -> Kernel version -> Custom Git repository
       -> URL of custom repository -> https://github.com/foss-for-synopsys-dwc-arc-processors/linux
       -> Custom repository version -> arc64
```

## Selecting a Linux Kernel Configuration File

You can select a Linux kernel configuration file in Buildroot configuration menu. For example:

```text
Kernel -> Defconfig name -> haps_hs
```

You can find a complete list of all Linux kernel configuration files in `arch/arc/configs` directory
of Linux kernel source tree:

```text
$ ls -1 <path-to-kernel-source-tree>/arch/arc/configs
axs101_defconfig
axs103_defconfig
axs103_smp_defconfig
haps_hs_defconfig
haps_hs_smp_defconfig
hsdk_defconfig
nsim_700_defconfig
nsimosci_defconfig
nsimosci_hs_defconfig
nsimosci_hs_smp_defconfig
tb10x_defconfig
vdk_hs38_defconfig
vdk_hs38_smp_defconfig
```

## Selecting a Prebuilt Toolchain

!!! info

    Each release of a toolchain may demand a specific set of options which must be selected/deselected manually.
    It happens, because when a custom toolchain is selected, Buildroot cannot detect toolchain's features automatically.
    Follow Buildroot's error messages for details.

By default, Buildroot builds a Linux toolchain for ARC from scratch and may use relatively old sources for tools like
GCC, Binutils, etc. If you want to use the latest available toolchain for ARC processors, then consider using
prebuilt components from [official releases page](https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain/releases).

In addition to the default values, select/deselect these options for the toolchain for ARC HS3x/4x with glibc library
(2023.03 release):

```text
Toolchain -> Toolchain type -> (X) External toolchain
          -> (X) Custom toolchain
          -> Toolchain origin -> (X) Toolchain to be downloaded and installed
          -> Toolchain URL -> https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain/releases/download/arc-2023.03-release/arc_gnu_2023.03_prebuilt_glibc_le_archs_linux_install.tar.gz
          -> External toolchain gcc version -> (X) 12.x
          -> External toolchain kernel headers series -> (X) 5.16.x
          -> External toolchain C library -> (X) glibc
          -> [ ] Toolchain has RPC support?
          -> [X] Toolchain has C++ support?
          -> [X] Toolchain has Fortran support?
```

Additional options for the toolchain for ARC HS3x/4x with uClibc library (2023.03 release):

```text
Toolchain -> Toolchain type -> (X) External toolchain
          -> (X) Custom toolchain
          -> Toolchain origin -> (X) Toolchain to be downloaded and installed
          -> Toolchain URL -> https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain/releases/download/arc-2023.03-release/arc_gnu_2023.03_prebuilt_uclibc_le_archs_linux_install.tar.gz
          -> External toolchain gcc version -> (X) 12.x
          -> External toolchain kernel headers series -> (X) 5.16.x
          -> External toolchain C library -> (X) uClibc/uClibc-ng
          -> [X] Toolchain has locale support?
          -> [ ] Toolchain has RPC support?
          -> [X] Toolchain has C++ support?
          -> [X] Toolchain has Fortran support?
```

## Selecting a Preinstalled Toolchain

A procedure is the same as for selecting a prebuilt toolchain except these ones:

```text
Toolchain -> Toolchain origin -> (X) Pre-installed toolchain
          -> Toolchain path -> /tools/toolchains/arc-linux-gnu
```

## Creating a Custom Configuration File

You can use `make savedefconfig` to save a current configuration to a configuration file.
It reduces all configuration options to a minimum set of options.

```shell
make savedefconfig DEFCONFIG=my_defconfig
```

Configuration file `my_defconfig` is saved in Buildroot's source tree. Here is an example of
a configuration file which selects a preinstalled toolchain:

```shell
BR2_arcle=y
BR2_archs38=y
BR2_TOOLCHAIN_EXTERNAL=y
BR2_TOOLCHAIN_EXTERNAL_CUSTOM=y
BR2_TOOLCHAIN_EXTERNAL_PATH="/tools/toolchains/arc-2023.03/arc-linux-gnu"
BR2_TOOLCHAIN_EXTERNAL_HEADERS_5_16=y
BR2_TOOLCHAIN_EXTERNAL_CUSTOM_GLIBC=y
# BR2_TOOLCHAIN_EXTERNAL_INET_RPC is not set
BR2_TOOLCHAIN_EXTERNAL_CXX=y
BR2_TOOLCHAIN_EXTERNAL_FORTRAN=y
BR2_TARGET_GENERIC_HOSTNAME="zebu_hs"
BR2_TARGET_GENERIC_ISSUE="Welcome to the HAPS Development Platform"
BR2_LINUX_KERNEL=y
BR2_LINUX_KERNEL_CUSTOM_VERSION=y
BR2_LINUX_KERNEL_CUSTOM_VERSION_VALUE="5.16"
BR2_LINUX_KERNEL_DEFCONFIG="haps_hs_smp"
BR2_LINUX_KERNEL_VMLINUX=y
BR2_TARGET_ROOTFS_INITRAMFS=y
```

## Using Custom Configuration Files

You can place a custom configuration file in `config` directory in Buildroot's source tree.
It allows you to use custom configuration files as usual:

```sgell
make my_defconfig
```

You can use `defconfig` command and `DEFCONFIG` environment variable to pass a path to the configuration file:

```shell
# Using an absolute path
$ make defconfig DEFCONFIG=/home/user/buildroot/my_defconfig

# Using a relative path (it's always relative to Buildroot's source tree)
$ make defconfig DEFCONFIG=my_defconfig
```

## Using Overlays

If you want to place addition files and directories in `rootfs`, then you can
use `BR2_ROOTFS_OVERLAY=<path>` parameter: it contains a path to the a directory
with files and directories which will be placed in the final filesystem image.

You can select it in a configuration menu:

```text
System configuration -> Root filesystem overlay directories -> overlay
```

Also you can achieve the same result by placing all necessary files and directories in `output/target`
(or just `target` if you build in a separate directory) directory and rebuilding images using `make`.
