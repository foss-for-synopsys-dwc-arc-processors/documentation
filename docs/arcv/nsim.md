# Running on nSIM

Consider this code example which prints all passed command
line arguments:

```c
#include <stdio.h>

int main(int argc, char *argv[]) {
    for (int i = 0; i < argc; i++) {
        printf("%d: %s\n", i, argv[i]);
    }
    return 0;
}
```

## Build and Run for Base RMX-100

Build the application:

```
$ riscv64-snps-elf-gcc \
        -march=rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr \
        -mabi=ilp32 \
        -mtune=arc-v-rmx-100-series \
        -specs=semihost.specs \
        -specs=arcv.specs \
        -T arcv.ld \
        args.c -o args.elf
```

Run on nSIM:

```
$ nsimdrv -p nsim_isa_family=rv32 \
          -p nsim_isa_ext=-all.i.c.zcb.zcmp.zcmt.zba.zbb.zbs.zicsr \
          -p nsim_semihosting=1 \
          -p enable_exceptions=0 \
          -- args.elf one two three
0: args.elf
1: one
2: two
3: three
```

## Build and Run for Base RMX-500

Build the application:

```
$ riscv64-snps-elf-gcc \
        -march=rv32ic_zcb_zcmp_zcmt_zba_zbb_zbs_zicsr \
        -mabi=ilp32 \
        -mtune=arc-v-rmx-500-series \
        -specs=semihost.specs \
        -specs=arcv.specs \
        -T arcv.ld \
        args.c -o args.elf
```

Run on nSIM:

```
$ nsimdrv -p nsim_isa_family=rv32 \
          -p nsim_isa_ext=-all.i.c.zcb.zcmp.zcmt.zba.zbb.zbs.zicsr \
          -p nsim_semihosting=1 \
          -p enable_exceptions=0 \
          -- args.elf one two three
0: args.elf
1: one
2: two
3: three
```

## Build and Run for Base RMX-500

Build the application:

```
$ riscv64-snps-elf-gcc \
        -march=rv32imac_zcb_zcmp_zba_zbb_zbs_zicsr \
        -mabi=ilp32 \
        -mtune=arc-v-rhx-100-series \
        -specs=semihost.specs \
        -specs=arcv.specs \
        -T arcv.ld \
        args.c -o args.elf
```

Run on nSIM:

```
$ nsimdrv -p nsim_isa_family=rv32 \
          -p nsim_isa_ext=-all.i.m.a.c.zcb.zcmp.zba.zbb.zbs.zicsr \
          -p nsim_semihosting=1 \
          -p enable_exceptions=0 \
          -- args.elf one two three
0: args.elf
1: one
2: two
3: three
```

## Build and Run for Base RPX-100

Build the application:

```
$ riscv64-snps-elf-gcc \
        -march=rv64imac_zcb_zba_zbb_zbs_zicsr \
        -mabi=lp64 \
        -mtune=arc-v-rmx-100-series \
        -mcmodel=medany \
        -Wl,-defsym=txtmem_addr=0x80000000 \
        -Wl,-defsym=txtmem_len=1M \
        -Wl,-defsym=datamem_addr=0x80200000 \
        -Wl,-defsym=datamem_len=1M \
        -specs=semihost.specs \
        -specs=arcv.specs \
        -T arcv.ld \
        args.c -o args.elf
```

Run on nSIM:

```
$ nsimdrv -p nsim_isa_family=rv64 \
          -p nsim_isa_ext=-all.i.m.a.c.zcb.zba.zbb.zbs.zicsr \
          -p nsim_semihosting=1 \
          -p enable_exceptions=0 \
          -- args.elf one two three
0: args.elf
1: one
2: two
3: three
```

## Debugging Using LLDB

For detailed information on how to debug an application with the Synopsys
LLDB Debugger and Visual Studio Code IDE, see the following page:
[The MetaWare Development Toolkit](https://foss-for-synopsys-dwc-arc-processors.github.io/arc-v-getting-started/synopsys-tools/mwdt.html).
