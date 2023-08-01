# Running QEMU Tests

TCG is the internal language that powers QEMU. There are some assembly tests that validate the
basic function of several instructions in QEMU.

Firstly, make sure that QEMU if configured with `--cross-cc-arc=arc-elf32-gcc` and `--cross-cc-arc64=arc64-elf-gcc` options.
Then after building QEMU use these commands to run TCG tests:

```shell
make clean-tcg
make build-tcg
make check-tcg
```
