# GSM-FR

## Introduction

Full Rate ([GSM-FR](http://www.quut.com/gsm/)) is the first digital speech
coding standard used in the GSM digital mobile phone system. The bit rate of
this codec is 13 kbit/s, or 1.625 bits/audio sample (often padded out to
33 bytes/20 ms or 13.2 kbit/s).

## Source Code

The source code is based on the original GSM-FR code ([3GPP TS 06.10](http://www.quut.com/gsm/))
created by Jutta Degener and Carsten Bormann, Technische Universitaet Berlin,
and then updated to 18 path level. The Application of this codec is to get
RPE-LTP from [G722](https://github.com/foss-for-synopsys-dwc-arc-processors/G722/tree/master/rpeltp),
commit [eddb5a231b51](https://github.com/foss-for-synopsys-dwc-arc-processors/G722/commit/eddb5a231b51),
and [G711](https://github.com/foss-for-synopsys-dwc-arc-processors/G722/tree/master/g711),
the same commit, for α-law and µ-law compression. Test sequences are taken from
[3GPP TS 06.10](http://www.3gpp.org/ftp/Specs/archive/06_series/06.10/0610-820.zip)
version 8.2.0 Release 99, June 2001, and this codec passes the test cases successfully.

## Multi-instance and Reentrance

Codec hasn't tested on multi-instance run, for more information about support
of multi-instance run see codec source documentation. Codec supports reentrance
in case if the value of `DSP_CTRL` register, AGU address pointer registers
(`AGU_AUX_APx`), AGU offset registers (`AGU_AUX_OSx`), AGU modifier registers
(`AGU_AUX_MODx`) and accumulators registers (`ACC0_LO`, `ACC0_HI`, `ACC0_GLO`
and `ACC0_GHI` in case of guard mode is on), refer **1** in [References](getting-started.md#references),
will be saved and restored.

## Codec-Specific Build Options

`LTO_BUILD` and `REMAPITU_T` options are enabled for this codec by default.

## Codec-Specific Run-Time Options

To run GSM-FR codec in MetaWare Debugger (refer **2** in
[References](getting-started.md#references)) use the following commands:

For GSM-FR codec:

```text
mdb -cl -run -nsim -tcf=<default TCF from /rules/common_hw_config.mk> \ 
    ARC_FR_CODEC_app.elf <number of frames> [-l|-u|-A] [-enc|-dec] \
    <input_file.inp> <output_file.cod> <BlockSize> <1stBlock> <NoOfBlocks>
```

Command-line options for GSM-FR encoder:

| Option | Description |
| --- | --- |
| `<input_file.inp>` | Specifies the input `*.inp` file |
| `<output_file.cod>` | Specifies the output `*.cod` file |
| `-l` | Input data for encoding and output data for decoding are in linear format (DEFAULT) |
| `-A` | Input data for encoding and output data for decoding are in α-law (G.711) format |
| `-u` | Input data for encoding and output data for decoding are in µ-law (G.711) format |
| `<BlockSize>` | Block size, in number of samples (default = 160) (optional) |
| `<1stBlock>` | Number of the first block of the input file to be processed (optional) |
| `<NoOfBlocks>` | Number of blocks to be processed, starting with block `<1stBlock>` |
| `<number of frames>` | Number of frames to be processed (optional) |
| `-enc` | Run only the encoder |

Command-line options for GSM-FR decoder:

| Option | Description |
| --- | --- |
| `<input_file.cod>` | Specifies the input `*.cod` file |
| `<output_file.out>` | Specifies the output `*.out` file |
| `-l` | Input data for encoding and output data for decoding are in linear format (DEFAULT) |
| `-A` | Input data for encoding and output data for decoding are in α-law (G.711) format |
| `-u` | Input data for encoding and output data for decoding are in µ-law (G.711) format |
| `<BlockSize>` | Block size, in number of samples (default = 160) (optional) |
| `<1stBlock>` | Number of the first block of the input file to be processed (optional) |
| `<NoOfBlocks>` | Number of blocks to be processed, starting with block `<1stBlock>` |
| `<number of frames>` | Number of frames to be processed (optional) |
| `-dec` | Run only the decoder |

## Examples

The following command encodes the linear `Seq01.inp` stream:

```text
mdb -cl -run -nsim -tcf=em9d_voice_audio ARC_FR_CODEC_app.elf -l \
    -enc ../testvectors/ref/Seq01.inp ../testvectors/Seq01.cod
```

The following command decodes the linear `Seq01.cod` stream without
any additional options:

```text
mdb -cl -run -nsim -tcf=em9d_voice_audio ARC_FR_CODEC_app.elf -l \
    -dec ../testvectors/ref/Seq01.cod ../testvectors/Seq01.out
```
