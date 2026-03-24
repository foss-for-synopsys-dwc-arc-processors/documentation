# LDAC Encoder

## Introduction

Lossless Digital Audio Coding technology ([LDAC](https://www.sony.net/Products/LDAC/?j-short=LDAC))
is a proprietary audio codec made by Sony used to stream high-quality music
to a Bluetooth device. This is a lossy codec that compresses audio with
different efficiency. LDAC supports three fixed rates: 330, 660, 990 kb/s and
an adaptive bitrate. Source code of LDAC encoder is available in Android Open
Source Project.

## Source Code

The original LDAC source code of the encoder was taken from [AOSP](https://android.googlesource.com/platform/external/libldac/+/refs/heads/master).
Version of the code taken: [commit](https://android.googlesource.com/platform/external/libldac/+/300aa818a1a6ee79), 20 February 2019.

## Multi-instance and Reentrance

Codec hasn't tested on multi-instance run, for more information about support
of multi-instance run see codec source documentation. Codec supports reentrance
in case if the value of `DSP_CTRL` register and accumulators registers
(`ACC0_LO`, `ACC0_HI`, `ACC0_GLO` and `ACC0_GHI` in case of guard mode is on),
refer **1** in [References](getting-started.md#references), will be saved and
restored.

## Codec-Specific Build Options

`LTO_BUILD` option is disabled for this codec by default. To build the LDAC
encoder, you must use this specific option: `FX_MAPPING=ON/OFF`. This option
turns on/off remapping of original mathematical fixed-point functions to the
optimized ones for ARCv2 DSP processor family. This option is enabled by default.

To build application and library with original (not optimized) mathematical
fixed-point functions, use the following command:

```text
gmake all FX_MAPPING=off
```

## Codec-Specific Run-Time Options

To run LDAC encoder in MetaWare Debugger (refer **2** in [References](getting-started.md#references)),
use the following command:

```text
mdb <mdb parameters> -tcf=<default TCF from /rules/common_hw_config.mk> \
    LDAC_ENCODER_app.elf -i <input_file> -o <output_file> -F <output_frame_size>
```

The following table lists parameters that can be passed to the LDAC encoder application:

| Option | Description | Available values |
| --- | --- | --- |
| `-i <input_file>` | Specifies the input WAV file | Any name of ``.wav` file without spaces |
| `-o <output_file>` | Specifies the output file | Any name of output file without spaces |
| `-F` | Set output frame size per one channel (in bytes) | 165, 110, 82, 66, 55, 47, 41, 36, 33, 30, 27, 25, 23 |

## Example

The following command runs the encoder in ARC nSIM simulator and encodes the
`.wav` file with a 110-byte output frame size:

```text
mdb -nsim -run -cl -tcf=em9d_voice_audio LDAC_ENCODER_app.elf -i example_96k_2byte_2ch.wav \
    -o example_96k_2byte_2ch_FS110.bin -F 110
```

Output file `example_96k_2byte_2ch_FS110.bin` must be bit-exact with the
similarly named reference file in the `ldac_encoder/testvectors/ref` folder.
