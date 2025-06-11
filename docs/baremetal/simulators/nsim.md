# nSIM

nSIM supports running and debugging applications built with GNU toolchain for all ARC families.

## Building and debugging applications

Suppose that `main.c` contains an application to be debugged on nSIM:

```c
int main()
{
    return 0;
}
```

Then build it (we use `-specs=nosys.specs` if input/output operations are not needed) for ARC HS3x using `-mcpu=hs38`:

```
$ arc-snps-elf-gcc -mcpu=hs38 -specs=nosys.specs -g main.c -o main.elf
```

Start nSIM with a GDB server with 12345 port:

```
$ nsimdrv -tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf -gdb -port 12345
```

Run GDB in another terminal and debug the application:

```
$ arc-snps-elf-gdb -quiet main.elf
Reading symbols from main.elf...
(gdb) target remote :12345
Remote debugging using :12345
0x80000000 in ?? ()
(gdb) load
Loading section .init, size 0x22 lma 0x100
Loading section .text, size 0x1548 lma 0x124
Loading section .fini, size 0x16 lma 0x166c
Loading section .ivt, size 0x54 lma 0x1682
Loading section .data, size 0x534 lma 0x36d8
Loading section .ctors, size 0x8 lma 0x3c0c
Loading section .dtors, size 0x8 lma 0x3c14
Loading section .sdata, size 0x10 lma 0x3c1c
Start address 0x00000124, load size 6952
Transfer rate: 188 KB/sec, 869 bytes/write.
(gdb) b main
Breakpoint 1 at 0x276: file main.c, line 3.
(gdb) c
Continuing.

Breakpoint 1, main () at main.c:3
3           return 0;
(gdb)
```

## Building and Running "Hello, World!"

Consider a simple example code (save it as `main.c`):

```c
#include <stdio.h>

int main()
{
    printf("Hello, World!\n");
    return 0;
}
```

You need to use `-specs=nsim.specs` to use input/output features and to pass `-on nsim_emt` option to nSIM
to use ARC GNU input/output protocol:

```
$ arc-snps-elf-gcc -mcpu=archs -specs=nsim.specs main.c -o main.elf
$ nsimdrv -tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf -on nsim_emt main.elf
Hello, World!
```

You can use MetaWare's own hostlink protocol for input/output operations by passing `-specs=hl.specs` to GCC.
In this case you don't have to pass any additional options to nSIM:

```
$ arc-snps-elf-gcc -mcpu=archs -specs=hl.specs main.c -o main.elf
$ nsimdrv -tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf main.elf
Hello, World!
```

See [Specs files](../general/specs.md) for detailed information about `.specs` files.

## Linking with size-optimized libraries

You can link your application with size-optimized versions of libraries
using `-specs=nano.specs`:

```
$ arc-snps-elf-gcc -mcpu=archs -specs=hl.specs -specs=nano.specs main.c -o main.elf
$ size main.elf
   text    data     bss     dec     hex filename
   5336    1564     472    7372    1ccc main.elf
```

Compare code size:

```
$ arc-snps-elf-gcc -mcpu=archs -specs=hl.specs main.c -o main.elf
$ size main.elf
   text    data     bss     dec     hex filename
  12396    2848     788   16032    3ea0 main.elf
```

## Building for big endian targets

To produce binaries for big-endian targets it's necessary to use `arceb-*`
toolchains (except ARCv3 toolchain which does not support big endian).
Also, you need to pass `-on nsim_isa_big_endian` to nSIM for big endian targets:

```
$ arceb-snps-elf-gcc -mcpu=archs -specs=nsim.specs main.c -o main.elf
$ nsimdrv -tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf -on nsim_emt -on nsim_isa_big_endian main.elf
Hello, World!
```

## Choosing nSIM templates

Different GCC options for ARC targets require different nSIM options for successful
execution of binaries. You can find them on corresponding pages for [ARCompact](../../toolchain/options/arcompact.md),
[ARCv2 HS](../../toolchain/options/arcv2.md#compatibility-for-predefined-targets-for-arc-hs),
[ARCv2 EM](../../toolchain/options/arcv2.md#compatibility-for-predefined-targets-for-arc-em) and
[ARCv3](../../toolchain/options/arcv3.md#compatibility-for-predefined-targets).

You can match `-mcpu` values for GCC with corresponding `-tcf` templates for nSIM using a table
below.

| Compiler             | `-mcpu`      | TCF             | Additional nSIM options                                                                                        |
|----------------------|--------------|-----------------|----------------------------------------------------------------------------------------------------------------|
| `arc-snps-elf-gcc`   | `hs`         | `hs36_base.tcf` |                                                                                                                |
| `arc-snps-elf-gcc`   | `hs34`       | `hs36.tcf`      |                                                                                                                |
| `arc-snps-elf-gcc`   | `hs38`       | `hs38_full.tcf` |                                                                                                                |
| `arc-snps-elf-gcc`   | `hs38_linux` | `hs38_full.tcf` | `-on nsim_isa_fpud_option -on nsim_isa_fpud_div_option -on nsim_isa_fpu_mac_option -on nsim_isa_fpu_hp_option` |
| `arc-snps-elf-gcc`   | `archs`      | `hs38_full.tcf` | `-p nsim_isa_mpy_option=2`                                                                                     |
| `arc-snps-elf-gcc`   | `em`         | `em6_mini.tcf`  | `-p nsim_isa_shift_option=3 -p nsim_isa_rgf_num_regs=32`                                                       |
| `arc-snps-elf-gcc`   | `em4`        | `em6_mini.tcf`  | `-p nsim_isa_shift_option=3 -p nsim_isa_rgf_num_regs=32`                                                       |
| `arc-snps-elf-gcc`   | `em4_dmips`  | `em6_dmips.tcf` |                                                                                                                |
| `arc-snps-elf-gcc`   | `em4_fpus`   | `em6_dmips.tcf` | `-on nsim_isa_fpus_option`                                                                                     |
| `arc-snps-elf-gcc`   | `em4_fpuda`  | `em6_dmips.tcf` | `-on nsim_isa_fpus_option -on nsim_isa_fpuda_option`                                                           |
| `arc-snps-elf-gcc`   | `arcem`      | `em6_dmips.tcf` | `-off nsim_isa_bitscan_option -off nsim_isa_div_rem_option`                                                    |
| `arc-snps-elf-gcc`   | `arc700`     | `arc770d.tcf`   |                                                                                                                |
| `arc-snps-elf-gcc`   | `arc600`     | `arc625d.tcf`   |                                                                                                                |
| `arc64-snps-elf-gcc` | `hs5x`       | `hs58_full.tcf` | `-on nsim_isa_ll64_option`                                                                                     |
| `arc64-snps-elf-gcc` | `hs58`       | `hs58_full.tcf` | `-on nsim_isa_ll64_option`                                                                                     |
| `arc64-snps-elf-gcc` | `hs6x`       | `hs68_full.tcf` |                                                                                                                |
| `arc64-snps-elf-gcc` | `hs68`       | `hs68_full.tcf` |                                                                                                                |

## Profiling and nCAM

You can run nSIM in NCAM mode - Near Cycle-Accurate Mode. This mode activates counters that depend on micro-architectural simulations. It may be a good tool for optimization and exploration. NCAM's model is not cycle-accurate and it's not derived from RTL, but it's much faster than xCAM. If you need a cycle-accurate model then consider using xCAM models.

Use `-on cycles` to enable NCAM:

```
$ nsimdrv -tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf -on cycles main.elf
```

Use `-on nsim_print_stats_on_exit` and `-on ncam_profiling` (this option enables more profiling counters) options to print simulation statistics at the end of a simulation:

```
$ arc-elf32-gcc -mcpu=archs -specs=hl.specs main.c -o main.elf
$ nsimdrv -tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf -on cycles -on ncam_profiling \
          -on nsim_print_stats_on_exit main.elf
<Main_Memory>
-------------------------------------------------------------------
 Main Memory          |            Frequency|                    %
 Read                 |                   86|                54.09
 Write                |                   73|                45.91
-------------------------------------------------------------------
 Total                |                  159|               100.00
</Main_Memory>
<L1-I-CACHE>
-------------------------------------------------------------------
 L1-I-CACHE           |            Frequency|                    %
-------------------------------------------------------------------
 Read Hits            |                  770|                94.48
 Read Misses          |                   45|                 5.52
-------------------------------------------------------------------
 Total                |                  815|               100.00
</L1-I-CACHE>
<L1-D-CACHE>
-------------------------------------------------------------------
 L1-D-CACHE           |            Frequency|                    %
-------------------------------------------------------------------
 RW Misses            |                   17|                 4.47
 RW Hits              |                  363|                95.53
 Read Hits            |                   89|                23.42
 Read Misses          |                    1|                 0.26
 Write Hits           |                  274|                94.48
 Write Misses         |                   16|                 4.21
 Dirty Misses         |                    0|                 0.00
-------------------------------------------------------------------
 Span Lines           |                    2|                 0.53
 Double Miss          |                    0|                 0.00
-------------------------------------------------------------------
 Total                |                  380|               100.00
</L1-D-CACHE>
<Statistics-Branch_Predictor_FB-GShare>
 Description: FB-GShare Branch Predictor Statistics
-------------------------------------------------------------------
 BPU (Two-Level)      |            Frequency|                    %
-------------------------------------------------------------------
 Correctly Predicted  |                  327|                76.05
 Miss Predicted       |                  103|                23.95
 Conditional Misses   |                   26|                25.24
 Uconditional Misses  |                   77|                74.76
-------------------------------------------------------------------
 Total                |                  430|               100.00
</Statistics-Branch_Predictor_FB-GShare>

<Histogram-Instructions>
-------------------------------------------------------------------
 Instruction          |            Frequency|                    %
-------------------------------------------------------------------
 stw                  |                  302|                25.44
 nop                  |                  197|                16.60
 mov                  |                  128|                10.78
...
 neg                  |                    1|                 0.08
 div                  |                    1|                 0.08
-------------------------------------------------------------------
 Delay Slot           |                   55|                 4.63
-------------------------------------------------------------------
 Total                |                 1187|               100.00
</Histogram-Instructions>
<Summary-Execution_Profile>
-------------------------------------------------------------------
 Execution Profile    |            Frequency|                    %
-------------------------------------------------------------------
 Interpreted Inst     |                 1187|               100.00
 Cond Branches        |                  102|                 8.59
 Cond Branch Mispred  |                   26|                 2.19
 Ucond Branches       |                  121|                10.19
 Ucond Branch Mispred |                   77|                 6.49
-------------------------------------------------------------------
 Total                |                 1187|               100.00
</Summary-Execution_Profile>
<Summary-Simulation_Time>
-------------------------------------------------------------------
 Simulation Time      |              Seconds|                    %
-------------------------------------------------------------------
 Simulation           |               0.0032|                99.72
 Hostlink             |               0.0000|                 0.28
-------------------------------------------------------------------
 Total                |               0.0032|               100.00
</Summary-Simulation_Time>
<Summary-Simulation_Performance>
 Instruction Count =   1187 [# Total]
 Simulation  Time  =   0.00 [Seconds]
 Simulation  Rate  =   0.38 [MIPS]
 Cycle Count       =   2766 [Cycles]
 CPI               =    2.330
 IPC               =    0.429
 Effective Clock   =   0.9 [MHz]
</Summary-Simulation_Performance>
```

Use `-on nsim_trace` and `-p nsim_trace-output=trace.txt` options to trace instructions (omit `nsim_trace-output` if you want to print trace log right into `stdout`):

```
$ nsimdrv -tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf -on nsim_trace \
          -p nsim_trace-output=trace.txt main.elf
$ head trace.txt

                nSIM, Version: 2023.03 (Build: 002)

[0x00000124] 0x226a0280                 K       lr             r2,[0xa] : (w0) r2 <= 0x00000000: aux[0x0a] => 0x00 *
[0x00000128] 0x224f04c2                 K       bset           r2,r2,0x13 : (w0) r2 <= 0x00080000 *
[0x0000012c] 0x20290080                 K       flag           r2 *
[0x00000130] 0x26ab740a 0x00000122   AD K       sr             00000122,0x290: aux[0x290] <= 0x122 *
[0x00000138] 0x220a3f80 0x00005c10   AD K       mov            gp,00005c10 : (w0) r26 <= 0x00005c10 *
[0x00000140] 0x42c3     0x00005b20   AD K       mov_s          r2,00005b20 : (w0) r2 <= 0x00005b20 *
[0x00000146] 0x26027083 0x00005e34   AD K       sub            r3,00005e34,r2 : (w0) r3 <= 0x00000314 *
```
