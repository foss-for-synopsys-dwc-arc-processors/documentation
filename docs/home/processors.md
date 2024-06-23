# Synopsys ARC Processors

## ARC Classic

Synopsys ARC Classic processors are represented by three generations of Instruction Set Architectures (ISA).
Each ISA consists of one or more processor families with different microarchitectures. All cores may be
configured to support little or big endianness. All families except ARC EM and ARC 600 families may be configured to support
running complex operation systems like Linux.

Here is a list of ARC processor families:

1. ARCompact ISA (also known as ARCv1 ISA):

    * [ARC 600 Processor Family of 32-bit cores](https://www.synopsys.com/designware-ip/processor-solutions/arc-600-family.html)
    * [ARC 700 Processor Family of 32-bit cores](https://www.synopsys.com/designware-ip/processor-solutions/arc-700-family.html)

2. ARCv2 ISA:

    * [ARC EM Processor Family of 32-bit cores](https://www.synopsys.com/designware-ip/processor-solutions/arc-em-family.html)
    * [ARC HS Processor Family of 32-bit cores (HS3x, HS4x, HS4xD)](https://www.synopsys.com/designware-ip/processor-solutions/arc-hs-family.html)

3. ARCv3 ISA:

    * [ARC HS Processor Family of 32-bit cores (HS5x, HS5xD)](https://www.synopsys.com/dw/ipdir.php?ds=arc-HS5x-processors)
    * [ARC HS Processor Family of 64-bit cores (HS6x)](https://www.synopsys.com/dw/ipdir.php?ds=arc-HS5x-processors)

4. [ARC VPX DSP Processors](https://www.synopsys.com/designware-ip/processor-solutions/arc-vpx-dsp-family.html) (not supported by GNU toolchain)

5. [ARC NPX Neural Processing Unit](https://www.synopsys.com/designware-ip/processor-solutions/arc-npx-family.html) (not supported by GNU toolchain)

You can find programmer's reference manuals and datasheets for each core on corresponding web pages.

## ARC-V

The Synopsys ARC-V processors leverage the proven microarchitecture of the
existing ARC processor offerings, while giving customers access to the
expanding RISC-V ecosystem. The Synopsys ARC-V portfolio, based on the
RISC-V ISA, includes high-performance, mid-range, and ultra-low power
families, as well as functional safety (FS) versions, to address a broad
range of application workloads.

A detailed overview of Synopsys ARC-V processors may be found on
[ARC-V Getting Started page](https://foss-for-synopsys-dwc-arc-processors.github.io/arc-v-getting-started/overview.html).
