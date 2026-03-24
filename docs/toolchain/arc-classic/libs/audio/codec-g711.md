# G.711

## Introduction

[G.711](https://www.itu.int/rec/recommendation.asp?lang=en&parent=T-REC-G.711-198811-I)
is an ITU-T standard for audio companding: Pulse code modulation (PCM) of voice frequencies.

## Source Code

This codec is based on the original [G.711](https://www.itu.int/rec/recommendation.asp?lang=en&parent=T-REC-G.711-198811-I),
25 November 1988, corresponding ANSI-C code is available in the G.711 module
of the ITU-T G.191 Software Tools Library, 13 January 2019, and this codec
passes the test cases successfully.

## Multi-instance and Reentrance

Codec hasn't tested on multi-instance run, for more information about support
of multi-instance run see codec source documentation. Codec supports reentrance
in case if AGU address pointer registers (`AGU_AUX_APx`), AGU offset registers
(`AGU_AUX_OSx`), AGU modifier registers (`AGU_AUX_MODx`), refer **1** in
[References](getting-started.md#references), will be saved and restored.

## Codec-Specific Build Options

`LTO_BUILD` and `REMAPITU_T` options are not available for this codec. Use the
following command to build the G.711 codec:

```text
mdb -run -cl -nsim -tcf=<default TCF from /rules/common_hw_config.mk> g711demo.elf \
    <number of frames> [u|A] [-r] [lili|loli|lilo] <input_file> <output_file> \
    <BlockSize> <1stBlock> <NoOfBlocks> [-skip n]
```

The following table lists the G.711 codec parameters:

| Option | Description |
| --- | --- |
| `<input_file>` | Specifies the input file |
| `<output_file>` | Specifies the output file |
| `-r` | Disables even-bit swap by A-law encoding and decoding (DEFAULT) |
| `A` | Input data for encoding and output data for decoding are in α-law (G.711) format |
| `u` | Input data for encoding and output data for decoding are in µ-law (G.711) format |
| `lili` | Convert the input file linear to linear: lin -> (A/u)log -> lin |
| `lilo` | Convert the input file linear to (A/u)-log |
| `loli` | Convert the input file (A/u) log to linear |
| `<BlockSize>` | Number of samples per block `[256]` (optional) |
| `<1stBlock>` | Number of the first block of the input file to be processed (optional) |
| `<NoOfBlocks>` | Number of blocks to process, starting with block `<1stBlock>` |
| `-skip n` | `n` is the number of samples to skip (optional) |

## Examples

The following command converts the linear samples to A-law log:

```text
mdb -run -cl -nsim -tcf=em9d_voice_audio g711demo.elf A lilo ../testvectors/Ref/sweep.src ../testvectors/sweep-r.a 256 1 256
```

The following command converts from A-law log to linear samples:

```text
mdb -run -cl -nsim -tcf=em9d_voice_audio g711demo.elf A loli ../testvectors/Ref/sweep-r.a ../testvectors/sweep.src 256 1 256
```
