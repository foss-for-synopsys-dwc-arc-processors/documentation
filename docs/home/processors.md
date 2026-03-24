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

### ARC-V RMX-100 and RMX-500 Series

The 32-bit ARC-V RMX series includes the three-stage-pipeline RMX-100 and the five-stage-pipeline RMX-500 processors,
which are optimized for ultra-low power embedded applications. The processors offer optional DSP support for greater
signal processing efficiency. The RMX-100 processor offers support for ISO 26262 functional-safety compliance
(QM and ASIL-D), and the RMX-500 processor offers support for functional-safety compliance (QM, ASIL-D, ASIL-B,
and hybrid mode), as well as ISO 21434 cybersecurity compliance.

More information is available on the Synopsys website:

* [Power-Efficient RISC-V Processors for Embedded Applications](https://www.synopsys.com/designware-ip/processor-solutions/arc-v-processors/arc-v-rmx.html)
* [Synopsys ARC-V RMX-100 Processor IP](https://www.synopsys.com/dw/ipdir.php?ds=arc-v-rmx-100)
* [Synopsys ARC-V RMX-500 Processor IP](https://www.synopsys.com/dw/ipdir.php?ds=arc-v-rmx-500)

### ARC-V RHX Series

The 32-bit ARC-V RHX-100 series consists of superscalar, single-core and multicore processors optimized
for efficient real-time applications. It includes support for coherent accelerators and real-time hardware
virtualization as well as optional RVV extensions (RHX-100V/105V). For safety-critical applications, the
RHX-100 series offers functional-safety-compliant versions of all the processors in the series.

More information is available on the Synopsys website:

* [Maximum Performance Efficiency for Real-time Applications](https://www.synopsys.com/designware-ip/processor-solutions/arc-v-processors/arc-v-rhx.html)
* [Synopsys ARC-V RHX-100 Processor IP](https://www.synopsys.com/dw/ipdir.php?ds=arc-v-rhx-100)

### ARC-V RPX Processors

The 64-bit ARC-V RPX-100 series consists of multicore processors with 64-bit RISC-V defined profile
support for both user and supervisor modes. The multicore ARC-V RPX-105 supports SMP Linux and shared
L3 cache. It offers configurations up to 16 cores and is optimized for efficient host processing
performance for a variety of applications. The RPX-100 series also offers functional-safety- compliant
versions of its processors.

More information is available on the Synopsys website:

* [Maximum Performance Efficiency for Host Applications](https://www.synopsys.com/designware-ip/processor-solutions/arc-v-processors/arc-v-rpx.html)
* [ARC-V RPX-100 Processor IP](https://www.synopsys.com/dw/ipdir.php?ds=arc-v-rpx-100)
