# G.726

## Introduction

[G.726](https://www.itu.int/rec/T-REC-G.191-201901-I/en) is an ITU-T ADPCM
speech codec standard covering the transmission of voice at rates of 16, 24,
32, and 40 kbit/s.

## Source Code

The original G.726 source code and test sequences are taken from [G.191](https://www.itu.int/rec/T-REC-G.191-201901-I/en)
version 2.0, 24 January 2000, and this codec passes the test cases successfully.

## Multi-instance and Reentrance

Codec hasn't tested on multi-instance run, for more information about support
of multi-instance run see codec source documentation. Codec supports reentrance
in case if AGU address pointer registers (`AGU_AUX_APx`), AGU offset registers
(`AGU_AUX_OSx`), AGU modifier registers (`AGU_AUX_MODx`), refer **1** in
[References](getting-started.md#references), will be saved and restored.

## Codec-Specific Build Options

`LTO_BUILD` and `REMAPITU_T` options are enabled for that codec by default.

## Codec-Specific Run-Time Options

To run G.726 codec in MetaWare Debugger (refer **2** in
[References](getting-started.md#references)), use the following commands.

With variable bitrate:

```text
mdb -run -cl -nsim -tcf=<default TCF from /rules/common_hw_config.mk> vbr-g726.elf \
    [-enc|-dec] -law [A|u|l] -rate [16|24|32|40|16-24-32-40] -frame [NoOfSamples] \
    [-noreset] [-?|-help] <InpFile> <OutFile> [<FrameSize> [<1stBlock> [<NoOfBlocks> [<Reset>]]]]
```

With constant bit rate:

```text
mdb -run -cl -nsim -tcf=<default TCF from /rules/common_hw_config.mk> g726demo.elf \
    [-noreset] [-?|-help] <Law> <Transf> <Rate>  <InpFile> <OutFile> \
    [<FrameSize> [<1stBlock> [<NoOfBlocks> [<Reset>]]]]
```

Command-line options descriptions for G.726 with variable bitrate support:

| Option | Description |
| --- | --- |
| `<input_file>` | Specifies the input file |
| `<output_file>` | Specifies the output file |
| `-enc` | Encode mode. Default: run encoder and decoder (optional) |
| `-dec` | Encode mode. Default: run encoder and decoder (optional) |
| `-law A|u` | The letters A or a for G.711 A-law, letter u for G.711 u-law, or letter l for linear. If linear is chosen, A-law is used to compress/expand samples to/from the G.726 routines.Default is A-law (optional) |
| `-rate` | Bit-rate (in kbit/s): 40, 32, 24 or 16 (in kbit/s) or a combination of them using dashes (For example, 32-24 or 16-24-32). Default is 32 kbit/s (optional) |
| `-frame` | Number of samples per frame for switching bit rates. Default is 16 samples (optional) |
| `-noreset` | Do not reset the encoder/decoder (optional) |
| `-?|-help` | Print help message (optional) |
| `FrameSize` | Frame size, in number of samples; The bitrate only changes at the boundaries of a frame. Default: 16 samples (optional) |
| `1stBlock` | The number of the first block of the input file to be processed. Default: first block (optional) |
| `NoOfBlocks` | The number of blocks to be processed, starting with block `1stBlock` (optional) |
| `Reset` | If specified as 1, the coder and decoder are reset at the very beginning of the processing. If 0, the processing starts with the variables in an unknown state. Default is 1 (reset ON) (optional) |

Command-line options descriptions for G.726 with constant bitrate support:

| Option | Description |
| --- | --- |
| `<input_file>` | Specifies the input file |
| `<output_file>` | Specifies the output file |
| `Transf` | Desired conversion on the input file (optional): [lolo], (A/u)log -> ADPCM -> (A/u) log[load], (A/u)log -> ADPCM[adlo], ADPCM -> (A/u) log |
| `Law` | Desired law (either A or u) (optional) |
| `Rate` | The bit-rate (in kbit/s): 40, 32, 24 or 16 (in kbit/s)or a combination of them using dashes (For example, 32-24 or 16-24-32). Default is 32 kbit/s (optional) |
| `-frame` | Number of samples per frame for switching bit rates. Default is 16 samples (optional) |
| `-noreset` | Do not reset the encoder/decoder (optional) |
| `-?|-help` | Print help message (optional) |
| `FrameSize` | The frame size, in number of samples; the bitrate only changes at the boundaries of a frame. Default: 16 samples (optional) |
| `1stBlock` | The number of the first block of the input file to be processed. Default: first block (optional) |
| `NoOfBlocks` | The number of blocks to be processed, starting with block `1stBlock` (optional) |
| `Reset` | If specified as 1, the coder and decoder are reset at the very beginning of the processing. If 0, the processing starts with the variables in an unknown state. Default is 1 (reset ON) (optional) |

## Examples

The following command instructs the codec encoding and decoding audio file
decoder to A-law log with variable bitrate:

```text
mdb -run -cl -nsim -tcf=em9d_voice_audio vbr-g726.elf -q -law A -rate 16-24-32-40-32-24 \
    ../testvectors/Ref/voice.src ../testvectors/voicvbra.tst
```

The following command instructs the codec to encode converting from A-law log
input file to ADPCM decoded file with frame size is 16 and 1 start block:

```text
mdb -run -cl -nsim -tcf=em9d_voice_audio g726demo.elf -q a load 16 \
    ../testvectors/Ref/nrm.a ../testvectors/rn16fa.i 16 1 1024
```
