# Running Linux on nSIM Using MetaWare Debugger

## Preface

It's possible to run a Linux kernel on nSIM simulator using MetaWare debugger. It's just another way of
running the kernel using nSIM. However, it allows to run not only UP configuration, but SMP configurations too.

## Running Single-Core ARC HS38 Linux Kernel

!!! info

    The kernel must be built with `haps_hs_defconfig` Linux configuration.

Passing all configurations manually:

```text
mdb -nsim -av2hs -prop=cpunum=0 -mmuv4 -Xrtc -Xatomic -Xtimer0 -Xtimer1 -Xmpyd -Xqmpyh -Xdiv_rem \
    -toggle=deadbeef=1 -prop=nsim_mem-dev=uart0,kind=dwuart,base=0xf0000000,irq=24 -prop=mmu_pagesize=8192 \
    -prop=mmu_super_pagesize=2097152 -prop=mmu_stlb_entries=16 -prop=mmu_ntlb_ways=4 -prop=mmu_ntlb_sets=128 \
    -icache=16384,64,2,o -dcache=16384,64,4,o -prop=nsim_isa_aps_feature=1 -prop=nsim_isa_num_actionpoints=4 \
    -prop=nsim_isa_ll64_option=1 -prop=nsim_isa_rtc_option=1 -prop=nsim_isa_core=3 -noprofile -run -cl vmlinux
```

Using a predefined TCF template:

```text
mdb -tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf \
    -prop=nsim_mem-dev=uart0,kind=dwuart,base=0xf0000000,irq=24 \
    -prop=nsim_isa_rtc_option=1 -nsim -noprofile -run -cl vmlinux
```

## Running Multi-Core ARC HS38 Linux Kernel

!!! info

    The kernel must be built with `haps_hs_smp_defconfig` Linux configuration.

!!! warning

    `nsim_connect_*` properties are not supported in older nSIM (prior K-2015.06).
    Please check the nSIM documentation to replace them with deprecated `nsim_mcip_*` properties.

Running 2 cores using a predefined TCF template:

```text
export NSIM_MULTICORE=1
mdb -pset=1 -psetname=core0 -prop=ident=0x00000050 -cmpd=soc \
    -tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf \
    -prop=nsim_mem-dev=uart0,kind=dwuart,base=0xf0000000,irq=0,use_connect=1 \
    -prop=nsim_connect=2 -prop=nsim_connect_idu=1 -prop=nsim_connect_gfrc=1 \
    -prop=nsim_connect_ici=1 -prop=isa_counters=1 -prop=nsim_isa_pct_counters=8 \
    -prop=nsim_isa_pct_interrupt=1 -prop=nsim_isa_pct_interrupt=1 \
    -noprofile vmlinux
mdb -tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf \
    -pset=2 -psetname=core1 -prop=ident=0x00000150 -cmpd=soc \
    -prop=nsim_mem-dev=uart0,kind=dwuart,base=0xf0000000,irq=0,use_connect=1 \
    -prop=nsim_connect=2 -prop=nsim_connect_idu=1 -prop=nsim_connect_gfrc=1 \
    -prop=nsim_connect_ici=1 -prop=isa_counters=1 -prop=nsim_isa_pct_counters=8 \
    -prop=nsim_isa_pct_interrupt=1 -prop=nsim_isa_pct_interrupt=1 \
    -noprofile vmlinux
mdb -multifiles=core0,core1 -cmpd=soc -run -cl
```

Running 4 cores using a predefined TCF template:

```shell
export NSIM_MULTICORE=1
VMLINUX="vmlinux"
COMMON="-tcf $NSIM_HOME/etc/tcf/templates/hs38_full.tcf -prop=nsim_mem-dev=uart0,kind=dwuart,base=0xf0000000,irq=0,use_connect=1 -prop=nsim_connect=2 -prop=nsim_connect_idu=1 -prop=nsim_connect_gfrc=1 -prop=nsim_connect_ici=1 -prop=isa_counters=1 -prop=nsim_isa_pct_counters=8 -prop=nsim_isa_pct_interrupt=1 -prop=nsim_isa_pct_interrupt=1 -noprofile"
mdb -pset=1 -psetname=core0 -prop=ident=0x00000050 -cmpd=soc $COMMON $VMLINUX
mdb -pset=2 -psetname=core1 -prop=ident=0x00000150 -cmpd=soc $COMMON $VMLINUX
mdb -pset=3 -psetname=core2 -prop=ident=0x00000250 -cmpd=soc $COMMON $VMLINUX
mdb -pset=4 -psetname=core3 -prop=ident=0x00000350 -cmpd=soc $COMMON $VMLINUX
mdb -multifiles=core0,core1,core2,core3 -cmpd=soc -run -cl
```

## Running ARC770 Linux Kernel

Passing all configurations manually:

```text
mdb -a7 -nsim -Xlib -prop=nsim_mmu=3 -Xtimer0 -Xtimer1 -icache=16384,64,2,o -dcache=32768,64,4,o \
    -prop=cpunum=0 -prop=nsim_mem-dev=uart0,kind=dwuart,base=0xf0000000,irq=24 -prop=nsim_sc_mem_range_end=0xc0fbffff \
    -prop=nsim_isa_atomic_option=1 -prop=nsim_isa_num_actionpoints=8 -prop=nsim_isa_aps_feature=1 -noprofile -OK -run -cl vmlinux
```
