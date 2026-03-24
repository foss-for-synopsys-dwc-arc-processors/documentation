# Overview

An audio codec is a hardware device or computer program capable of encoding or
decoding a digital audio signal including music and human speech. In software
terms, an audio codec is a computer program implementing an algorithm that
compresses and decompresses digital audio data.

Telecommunications codecs had written using ITU-T/ETSI BASOPs for elementary
operations require bit-exact operations.

The objective of the algorithm is to represent the audio signal with minimum
number of bits while retaining acceptable perceptual quality characteristics.
This reduces the storage or the bandwidth requirements to store or transmit
digital audio. The embARC Audio codecs are set of software libraries
implementing legacy and commonly used codecs standardized by ETSI/3GPP and ITU-T.

DSP algorithms can be implemented using different data types, can come from
different sources, can have different requirements and can evolve differently.
Some usual cases are:

* Porting of integer C code using standard C integer types
* Porting of C code using floating-point (either using a floating-point unit or emulation)
* Porting fixed-point algorithms written with C integer types and operations
* Porting fixed-point algorithms written using special macros for fixed-point
  multiplication, multiply-add operations, and so on

## Codecs Supported by embARC Audio

### ETSI/3GPP Codecs

The corresponding page:

* [GSM-RF](codec-gsm-fr.md)

Codecs of this group are designed to transmit speech over a cellular network.
GSM Full Rate (GSM-FR) was the first digital speech coding standard used in
early deployments of GSM digital mobile phone systems. The GSM-FR bitrate is
13 kbps and has very moderate MIPS/memory requirements.

### ITU-T Codecs

The corresponding pages:

* [G.711](codec-g711.md)
* [G.722](codec-g722.md)
* [G.726](codec-g726.md)

These codecs are implementations of [G series of ITU-T standards](https://www.itu.int/rec/T-REC-G/en).
They are designed for voice transmission over Plain Switched Telephone Networks,
Voice over IP, and mobile networks.

### Bluetooth CVSD

The corresponding page:

* [BT-CVSD](codec-bt-cvsd.md)

Continuously Variable Slope Delta Modulation (BT-CVSD) is a mandatory codec for
Bluetooth to transmit speech at 8 kHz sampling rate at 64 kbps.

### Sony LDAC

The corresponding page:

* [LDAC](codec-ldac.md)

LDAC is an audio coding technology developed by [Sony](https://www.sony.net/Products/LDAC/),
which allows streaming audio over Bluetooth connections up to 990 kbps at
24 bit/96 kHz (also called high-resolution audio). It is used by various Sony
products.

## Optimizations

### ITU-T Basop Optimization

To improve performance of ITU-T and ETSI/3GPP codecs at early optimization step,
use Synopsys MetaWare Development Tools which efficiently replace
[ITU-T G.191 STL basic operators](https://www.itu.int/rec/T-REC-G.191/en) with
ARCv2 fixed point intrinsics at compile time.

### Link Time Optimization

Link Time Optimization (LTO) is an optimization technique of intermodular
optimization performed during the link stage.

### Compiler Optimization Options

To increase the performance of the code, use the following compiler optimizing
options:

* `Hinline_threshold`
* `Haggressive_unroll`
* `Hmisched`
* `Hunswitch`
* `Hmax_predicate`
* `Hdense_prologue`
* `fslp-vectorize-aggressive`
* `funroll-loops`
* `Hon=Zd_loops`
* `Hoff=Zd_loops`
* `Hinlsize`
* `Hunroll`
* `Hloop_sms`

For more details, refer **1** in [References](getting-started.md#references).

### Hmerge

This optimization option is similar to LTO. The difference is `Hmerge` occurs
at the compilation stage, not at the linking stage. In this case, all the
source code is collected in a single object file.
