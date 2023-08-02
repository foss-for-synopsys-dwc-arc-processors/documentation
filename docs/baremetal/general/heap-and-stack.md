# Heap and Stack Size

To change size of heap in baremetal applications the following option should be specified to the linker:
`--defsym=__DEFAULT_HEAP_SIZE=${SIZE}`, where `${SIZE}` is desired heap size, in bytes. It also possible to use
size suffixes, like `k` and `m` to specify size in kilobytes and megabytes respectively. For stack size respective
option is `--defsym=__DEFAULT_STACK_SIZE=${STACK_SIZE}`. Note that those are linker commands - they are valid only
when passed to `ld` application, if `gcc` driver is used for linking, then those options should be prefixed with
`-Wl`. For example:

```shell
$ arc-elf32-gcc -Wl,--defsym=__DEFAULT_HEAP_SIZE=256m \
  -Wl,--defsym=__DEFAULT_STACK_SIZE=1024m --specs=nosys.specs \
  hello.o -o hello.bin
```

Those options are valid only when default linker script is used. If custom linker script is used, then effective
way to change stack/heap size depends on properties of that linker script - it might be the same, or it might be
different.
