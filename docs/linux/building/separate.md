# Building Linux kernel and Filesystem Separately

## Preface

Sometimes it's more convenient to build root file system and a Linux kernel separately.
This guide covers an example when it's necessary to build those images separately
for ARC HS38 target.

Suppose, that a glibc-based toolchain for ARC HS38 is preinstalled in
`/tools/toolchains/arc-linux-gnu`.

## Building Root Filesystem

Clone Buildroot:

```shell
# Clone the latest Buildroot
git clone https://git.busybox.net/buildroot
# ... or use a custom repository for support of ARCv3 targets
git clone -b arc-2024.12 https://github.com/foss-for-synopsys-dwc-arc-processors/buildroot
cd buildroot
```

Use `snps_archs38_haps_defconfig` configuration file for this example and enter
a configuration menu:

```shell
make snps_archs38_haps_defconfig
make menuconfig
```

Deselect an option for the Linux kernel and select an appropriate filesystem
for your needs:

```text
Kernel -> [ ] Linux kernel
Filesystem images -> [*] cpio the root filesystem # For use as an initial RAM filesystem
                  -> ext2/3/4 root filesystem     # For mounting from a storage
```

Choose a preinstalled toolchain (a list of available features may differ
depending on a version of a toolchain, in case of errors follow Buildroot's
recommendations in error messages):

```text
Toolchain -> Toolchain type -> (X) External toolchain
          -> (X) Custom toolchain
          -> Toolchain origin -> (X) Pre-installed toolchain
          -> Toolchain path -> /tools/toolchains/arc-linux-gnu
          -> External toolchain gcc version -> (X) 14.x
          -> External toolchain kernel headers series -> (X) 5.16.x
          -> External toolchain C library -> (X) glibc
          -> [ ] Toolchain has RPC support?
          -> [X] Toolchain has C++ support?
          -> [X] Toolchain has Fortran support?
```

Build the filesystem image:

```shell
make
```

## Building the Linux Kernel Without Initial Root Filesystem

Clone a Linux kernel repository and choose a version you want:

```shell
git clone https://github.com/torvalds/linux
cd linux
git checkout v6.4
```

Configure the kernel:

```shell
make ARCH=arc haps_hs_defconfig
```

Put the toolchain to `PATH` environment variable and build the kernel:

```shell
export PATH=/tools/toolchains/arc-linux-gnu:$PATH
make ARCH=arc CROSS_COMPILE=arc-linux-
```

The Linux kernel is saved as `vmlinux` file.

## Building the Linux Kernel With Initial Root Filesystem

To link the Linux kernel with previously built root filesystem image (with `.cpio` extension), it's necessary to
set `CONFIG_INITRAMFS_SOURCE` options for the kernel. Configure the kernel and enter a configuration menu:

```shell
make ARCH=arc haps_hs_defconfig
make ARCH=arc menuconfig
```

Fill `CONFIG_INITRAMFS_SOURCE` parameter with a path to `rootfs.cpio`, which was built previously:

```text
General setup -> Initial RAM filesystem and RAM disk support
                 -> Initramfs source file(s) -> <Buildroot-source-tree>/output/images/rootfs.cpio
```

Put the toolchain to `PATH` environment variable and build the kernel:

```shell
export PATH=/tools/toolchains/arc-linux-gnu:$PATH
make ARCH=arc CROSS_COMPILE=arc-linux-
```

The Linux kernel with root file system is saved as `vmlinux` file.
