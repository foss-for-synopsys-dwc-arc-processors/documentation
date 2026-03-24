# G.722 (Appendix IV)

## Introduction

[G.722](https://www.itu.int/rec/T-REC-G.722) is an ITU-T standard 7 kHz
Wideband audio codec operating at 48, 56 and 64 kbit/s.

## Source Code

G.722 codec, with appendix IV for the decoder, and test sequences for use with
G.722 codec + G.722 decoder appendix IV were taken from
[ITU-T G.722](https://www.itu.int/rec/T-REC-G.722-201209-I/en) Software Release
3.00, October 2012 and from [G.191](https://www.itu.int/rec/T-REC-G.191-201901-I/en)
Software Tools Library, 13 January 2019, also, this codec passes the test cases successfully.

## Multi-instance and Reentrance

Codec hasn't tested on multi-instance run, for more information about support
of multi-instance run see codec source documentation. Codec supports reentrance
in case if the value of `DSP_CTRL` register, AGU address pointer registers
(`AGU_AUX_APx`), AGU offset registers (`AGU_AUX_OSx`), AGU modifier registers
(`AGU_AUX_MODx`) and accumulators registers (`ACC0_LO`, `ACC0_HI`, `ACC0_GLO`
and `ACC0_GHI` in case of guard mode is on), refer **1** in [References](getting-started.md#references),
will be saved and restored. Also if you build with option `USE_XCCM=on` you must
not use XCCM, because XCCM bank is used for storing table coefficients.

## Codec Specific Build Options

!!! warning

    XCCM bank is used for storing table coefficients.

`LTO_BUILD`, `USE_XCCM` and `REMAPITU_T` options are enabled for this codec
by default.

The following table lists specific build options of G.722 codec:

| Command | Description |
| --- | --- |
| `gmake COMPONENT=\<ENCODER/DECODER\>` | These commands build only encoder (`encg722.elf`) or decoder (`decg722.elf`) |
| `WMOPS=on` | Display information about each frame, frame number, total Weighted MOPS, computational complexity of the encoder or decoder for that frame, the average WMOPS figure for the frames processed, the observed worst case WMOPS figure and the observed "worst worst case" figure of the encoder or decoder for the current frame |
| `USE_XCCM=on` | Use XCCM bank for storing tables of coefficients in `funcg722.c` |

The following command builds the encoder application and library with WMOPS:

```text
gmake COMPONENT=ENCODER WMOPS=on
```

## Codec-Specific Run-Time Options

To run G.722 codec in MetaWare Debugger (refer **2** in
[References](getting-started.md#references)), use the following commands.

For G.722 encoder:

```text
mdb -run -cl -nsim -tcf=<default TCF from /rules/common_hw_config.mk> encg722.elf \
    [-q] [-mode <M>] [-byte] [-fsize N] [-frames N2] <file.inp> <file.out>
```

For G.722 decoder, appendix IV:

```text
mdb -run -cl -nsim -tcf=<default TCF from /rules/common_hw_config.mk> decg722.elf \
    [-fsize N] <file.inp> <file.out>
```

For G.722 test encoder:

```text
mdb -run -cl -nsim -tcf=<default TCF from /rules/common_hw_config.mk> tstcg722.elf \
    <file.inp> <file.ref>
```

For G.722 test decoder:

```text
mdb -run -cl -nsim -tcf=<default TCF from /rules/common_hw_config.mk> tstdg722.elf \
    <file.inp> <low_file.ref> <high_file.ref>
```

Command-line options descriptions for G.722 encoder:

| Option | Description |
| --- | --- |
| `<file.inp>` | Specifies the input `*.bin` file |
| `<file.out>` | Specifies the output `*.cod` file |
| `-byte` | Provide encoder output data in legacy byte-oriented format (default is `g192`) |
| `-fsize` | Number of 16 kHz input samples per frame (must be an even number). Default is 160 samples(16 kHz) (10 ms) |
| `-frames` | Number of frames to process (values -1 or 0 processes the whole file) |
| `-mode <M>` | Operating mode (1,2,3) (or rate 64, 56, 48 in kbps). Default is mode 1 (= 64 kbps) |
| `-h/-help` | Print help message |
| `-q` | Suppress debug information |

Command-line options descriptions for G.722 decoder of appendix IV:

| Option | Description |
| --- | --- |
| `<file.inp>` | Specifies the input file `*.bst` file |
| `<file.out>` | Specifies the output file `*.out` file |
| `-fsize` | Define frame size for `g192` operation and file reading |

Command-line options descriptions for test G.722 encoder:

| Option | Description |
| --- | --- |
| `<file.inp>` | Specifies the input file `*.cod` file |
| `<file.ref>` | Specifies the reference file for checking the correctness of Encoder |

Command-line options descriptions for test G.722 decoder:

| Option | Description |
| --- | --- |
| `<file.inp>` | Specifies the input file `*.cod` file |
| `<low_file.ref>` | Specifies the low part of reference file |
| `<high_file.ref>` | Specifies the high part of reference file |

## Examples

!!! note

    The decoder supports only `g192` byte format. For this reason, if you will
    encode a file in the legacy byte-oriented format using `-byte` option, you
    can not decode this file by the decoder.

The following command encodes the `inpsp.bin` stream to legacy byte-oriented format:

```text
mdb -run -cl -nsim -tcf=em9d_voice_audio encg722.elf -byte ../testvectors/inp/inpsp.bin ../testvectors/temp.cod
```

The following command decodes the `test10.bst` stream without any additional options:

```text
mdb -run -cl -nsim -tcf=em9d_voice_audio decg722.elf ../testvectors/inp/test10.bst ../testvectors/test10.out
```
