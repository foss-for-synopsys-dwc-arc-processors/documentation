# Running Linux on VDK

!!! warning

    There is no guarantee that this guide will be applicable for newer versions
    of a Linux kernel or a toolchain.

## Preface

This guide is tested on VDK HS38x4 2020.09. VDK itself requires nSIM Pro license.
ARC GNU toolchain 2017.09 based on uClibc-ng library is used for building a Linux kernel.
Ubuntu 18.04 is used as a host system.

## Prerequisites

Install packages for Ubuntu 18.04:

```shell
sudo apt install xterm libgl1-mesa-dri libsdl1.2debian
```

## Build Linux Kernel and Filesystem Image

Clone the latest Buildroot:

```shell
git clone -b 2024.11.1 https://git.busybox.net/buildroot
cd buildroot
```

Filesystem images must not be linked against Linux kernel for running on VDK. Also,
`vdk_hs38_smp` configuration file must be used for the Linux kernel. Create
a `vdk_defconfig` configuration file in `configs` directory with a correct configuration:

```shell
BR2_arcle=y
BR2_archs38=y
BR2_TOOLCHAIN_EXTERNAL=y
BR2_TOOLCHAIN_EXTERNAL_CUSTOM=y
BR2_TOOLCHAIN_EXTERNAL_DOWNLOAD=y
BR2_TOOLCHAIN_EXTERNAL_URL="https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain/releases/download/arc-2017.09-release/arc_gnu_2017.09_prebuilt_uclibc_le_archs_linux_install.tar.gz"
BR2_TOOLCHAIN_EXTERNAL_GCC_7=y
BR2_TOOLCHAIN_EXTERNAL_HEADERS_4_12=y
BR2_TOOLCHAIN_EXTERNAL_LOCALE=y
BR2_TOOLCHAIN_EXTERNAL_HAS_SSP=y
BR2_TOOLCHAIN_EXTERNAL_CXX=y
BR2_LINUX_KERNEL=y
BR2_LINUX_KERNEL_CUSTOM_VERSION=y
BR2_LINUX_KERNEL_CUSTOM_VERSION_VALUE="4.9"
BR2_LINUX_KERNEL_DEFCONFIG="vdk_hs38_smp"
BR2_LINUX_KERNEL_VMLINUX=y
BR2_TARGET_ROOTFS_EXT2=y
```

Build images:

```shell
make vdk_defconfig
make
```

Linux kernel and filesystem images reside in `output/images` now:

```text
$ ls output/images/
rootfs.ext2  rootfs.tar  vmlinux
```

## Running Linux

ARC HS VDK already includes Linux kernel image and root file system image.
Save old images and replace them with your newly generated files:

```shell
cd <VDK-directory>/skins/ARC-Linux
mv rootfs.ARCv2.ext2{,.orig}
cp <Buildroot-source-tree>/output/images/rootfs.ext2 rootfs.ARCv2.ext2
mv ARCv2/vmlinux_smp-noSD{,.orig}
cp <Buildroot-source-tree>/output/images/vmlinux ARCv2/vmlinux_smp-noSD
```

Run a simulator:

```shell
<VDK-directory>/skins/ARC-Linux/start_interactive.tcl
```

## Configuring Network

Before running VDK if you wish to have a working networking connection on Linux for ARC system it is required to configure VDK VHub application.
By default this application will pass all Ethernet packets to the VDK Ethernet model, however on busy networks that can be too much to
handle in a model, therefore it is highly recommended to configure destination address filtering. Modify `VirtualAndRealWorldIO/VHub/vhub.conf`:
set `DestMACFilterEnable` to `true`, and append some random valid MAC address to the list of `DestMACFilter`, or use one of the MAC address
examples in the list. This guide will use `D8:D3:85:CF:D5:CE` - this address is already in the list. Note that is has been observed that
it is not possible to assign some addresses to Ethernet device model in VDK, instead of success there is an error “Cannot assign requested address”.

Note, that due to the way how VHub application works, it is impossible to connect to the Ethernet model from the host on which it runs
on and vice versa. Therefore to use networking in target it is required to either have another host and communicate with it.

Run VHub application as `root`:

```shell
<VDK-directory>/VirtualAndRealWorldIO/VHub/vhub -f VirtualAndRealWorldIO/VHub/vhub.conf
```

In another console launch VDK as a regular user:

```shell
<VDK-directory>/skins/ARC-Linux/start_interactive.tcl
```

After VDK will load, start simulation. After Linux kernel will boot, login into system via UART console:
login root, no password. By default networking is switched off. Enable `eth0` device, configure it is use MAC
from address configured in VHub:

```shell
ifconfig eth0 hw ether d8:d3:85:cf:d5:ce
ifconfig eth0 up
```

Linux kernel will emit errors about failed PTP initialization - those are expected.
Assign IP address to the target system. This example uses DHCP:

```shell
udhcpc eth0
```

Now it is possible to mount some NFS share and run applications from it:

```shell
mount -t nfs public-nfs:/home/arc_user/pub /mnt
/mnt/hello_world
```
