# Baremetal Targets

!!! warning

    Note that Ashling Opella-XD is EOL (End of Life) and Eclipse IDE may not
    support the latest versions of the probe. Refer
    [the old documentation site](https://foss-for-synopsys-dwc-arc-processors.github.io/toolchain/)
    for guides related to Ashling Opella-XD.

This sections is related to building and running applications for baremetal
targets (without an operating system) using `arc-elf32` (ARCompact or ARCv2)
and `arc64-elf` (ARCv3) toolchains with Newlib standard library. Such binaries
may be run on these platforms:

1. Simulators: nSIM and QEMU.
2. Development platforms: HS Development Kit, EM Software Development Platform.
   IoT Development Kit.
3. Guides for EM Starter Kit and AXS Software Development
   Platform are presented too but these platform are no longer supported and
   there is no guarantee that those guides will be applicable for the latest
   tools.
