# Testing Specific RISC-V Extensions

One of the benefits of RISC-V ISA is its extensibility. As a matter of fact,
only a handful of instructions form the minimally required base. But the most
interesting part is in utilizing application-specific extensions, which let
you achieve benefits in certain situations.

In embedded and especially deeply embedded applications, one of the key objectives
is to keep the memory footprint of the target application as tiny as possible. And
the RISC-V `Zc*` extensions help to achieve exactly that. This section demonstrates
a couple of those extensions in action. See also the documentation and related
information on each extension on the corresponding GitHub resource here:
https://github.com/riscv/riscv-code-size-reduction.


## Zcmp Extension

Consider the following code snippet:

```
$ cat t01.c

void test (void) {
    asm volatile (""
             :
             :
             : "s0", "s1", "s2", "s3",
               "s4", "s5", "s6", "s7",
               "s8", "s9", "s10", "s11");
}
```

Here is a generated assembly listing targeting the `Zcmp` extension:

```
$ riscv64-snps-elf-gcc -march=rv32im_zcmp -mabi=ilp32 t01.c -S -Os -o -

        .file   "t01.c"
        .option nopic
        .attribute arch, "rv32i2p1_m2p0_zca1p0_zcmp1p0"
        .attribute unaligned_access, 1
        .attribute stack_align, 16
        .text
        .align  1
        .globl  test
        .type   test, @function
test:
        cm.push {ra, s0-s11}, -64
        cm.popret       {ra, s0-s11}, 64
        .size   test, .-test
        .ident  "GCC: (RISC-V elf toolchain - build 8093) 14.2.0"
        .section        .note.GNU-stack,"",@progbits
```

Note the generated `cm.push` and `cm.popret` instructions, which confirm that
the compiler is capable of using the extension.

## Zcb Extension

Another RISC-V extension that is easy to see in action also comes from the
code-size-reduction group: `Zcb`. Consider the following C function:

```
$ cat t03.c

int test (int a, int b) {
    return a + b;
}
```

Now compile and disassemble it:

```
$ riscv64-snps-elf-gcc -march=rv32im_zca_zcb -mabi=ilp32 t03.c -c -O1
$ riscv64-snps-elf-objdump -d t03.o

t03.o:     file format elf32-littleriscv


Disassembly of section .text:

00000000 <test>:
   0:   952e                    add     a0,a0,a1
   2:   8082                    ret
```

Note the short 16-bit encoding of the add instruction. Without use of `Zc` extensions, compiler
generates a full 32-bit encoding as in the example below.

```
$ riscv64-snps-elf-gcc -march=rv32im -mabi=ilp32 t03.c -c -O1
$ riscv64-snps-elf-objdump -d t03.o

t03.o:     file format elf32-littleriscv


Disassembly of section .text:

00000000 <test>:
   0:   00b50533                add     a0,a0,a1
   4:   00008067                ret
```
