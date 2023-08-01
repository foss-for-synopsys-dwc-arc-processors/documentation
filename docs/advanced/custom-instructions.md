# Custom instructions

## Extension Instructions Encoding

ARC architecture allows users to specify extension instructions. These extension instructions are not
macros. The assembler creates encodings for using these instructions according to the specification by the user.
Here is a list of supported instructions:

| Number of operands | Major opcode | Sub-opcode 1             | Sub-opcode 2 | Sub-opcode 3 |
|--------------------|--------------|--------------------------|--------------|--------------|
| 3                  | `0x07`       | `0x00-0x2E`, `0x30-0x3f` | —            | —            |
| 2                  | `0x07`       | `0x2F`                   | `0x00-0x3E`  | —            |
| 1                  | `0x07`       | `0x2F`                   | `0x3F`       | `0x00-0x3F`  |
| 0                  | `0x07`       | `0x2F`                   | `0x3F`       | `0x00-0x3F`  |

### Three-operand Instructions

Three-operand instructions have `op<.cc><.f> a,b,c` syntax format, and it is the most
general form of an ARC instruction:

```text
op<.f> a,b,c
op<.f> a,b,u6
op<.f> b,b,s12
op<.cc><.f> b,b,c
op<.cc><.f> b,b,u6
op<.f> a,limm,c
op<.f> a,limm,u6
op<.f> 0,limm,s12
op<.cc><.f> 0,limm,c
op<.cc><.f> 0,limm,u6
op<.f> a,b,limm
op<.cc><.f> b,b,limm
op<.f> a,limm,limm
op<.cc><.f> 0,limm,limm
op<.f> 0,b,c
op<.f> 0,b,u6
op<.f> 0,limm,c
op<.f> 0,limm,u6
op<.f> 0,b,limm
op<.f> 0,limm,limm
```

### Two-operand Instructions

Two-operand instructions have the following syntax format:

```text
op<.f> b,c
op<.f> b,u6
op<.f> b,limm
op<.f> 0,c
op<.f> 0,u6
op<.f> 0,limm
```

### One-operand and Zero-operand Instructions

One-operand instructions are having the following syntax format:

```text
op<.f> c
op<.f> u6
op<.f> limm
```

Zero-operand instructions are actually has `op<.f> u6` one-operand instruction syntax, with `u6` set to zero.

To create a custom instruction, ones need to make use of the `.extInstruction` pseudo-op, which
also allows the user to choose for a particular instruction syntax:

## Defining Custom Instructions

`.extInstruction` pseudo-op is used to create a custom instruction. On top of the formal syntax choices,
we have also syntax class modifiers:

* `OP1_MUST_BE_IMM` which applies for `SYNTAX_3OP` type of extension instructions, specifying that the first
  operand of a three-operand instruction must be an immediate (i.e., the result is discarded). This is
  usually used to set the flags using specific instructions and not retain results.
* `OP1_IMM_IMPLIED` modifies syntax class `SYNTAX_2OP`, specifying that there is an implied
  immediate destination operand which does not appear in the syntax. In fact this is actually
  an 3-operand encoded instruction!

### Example №1. Three-operand instruction

Definitions of the instruction:

```gas
.extInstruction insn1, 0x07, 0x2d, SUFFIX_NONE, SYNTAX_3OP|OP1_MUST_BE_IMM
```

Using the instruction:

```gas
insn1  0,b,c
insn1  0,b,u6
insn1  0,limm,c
insn1  0,b,limm
```

### Example №2. Two-operand instruction

!!! note

    The encoding of `insn2` uses the `SYNTAX_3OP` format (i.e., Major `0x07` and SubOpcode1: `0x00-0x2E`, `0x30-0x3F`)

Definitions of the instruction:

```gas
.extInstruction insn2, 0x07, 0x2d, SUFFIX_NONE, SYNTAX_2OP|OP1_IMM_IMPLIED
```

Using the instruction:

```gas
insn2  b,c
insn2  b,u6
insn2  limm,c
insn2  b,limm
```

### Example №3. All Variants

Definitions of the instructions:

```gas
.extInstruction insn1, 7, 0x21, SUFFIX_NONE, SYNTAX_3OP
.extInstruction insn2, 7, 0x21, SUFFIX_NONE, SYNTAX_2OP
.extInstruction insn3, 7, 0x21, SUFFIX_NONE, SYNTAX_1OP
.extInstruction insn4, 7, 0x21, SUFFIX_NONE, SYNTAX_NOP
```

Using the instruction:

```gas
start:
    insn1   r0,r1,r2
    insn2   r0,r1
    insn3   r1
    insn4
```

Disassembly of section `.text`:

```objdump
0x0000 <start>:
   0:   3921 0080               insn1   r0,r1,r2
   4:   382f 0061               insn2   r0,r1
   8:   392f 407f               insn3   r1
   c:   396f 403f               insn4
```

## Support in GCC

### Using Inline Functions

Define a macro for a two-operand custom instruction:

```c
#define intrinsic_2OP(NAME, MOP, SOP)       \
    ".extInstruction " NAME "," #MOP ","    \
    #SOP ",SUFFIX_NONE, SYNTAX_2OP\n\t"
```

Instantiate the custom instruction:

```c
__asm__ (intrinsic_2OP ("chk_pkt", 0x07, 0x01));
```

Create an inline function:

```c
__extension__ static __inline int32_t __attribute__ ((__always_inline__))
__chk_pkt (int32_t __a)
{
  int32_t __dst;
  __asm__ ("chk_pkt %0, %1\n\t"
             : "=r" (__dst)
             : "rCal" (__a));
  return __dst;
}
```

Here is a full example:

```c
#include <stdint.h>

#define intrinsic_2OP(NAME, MOP, SOP)       \
    ".extInstruction " NAME "," #MOP ","    \
    #SOP ",SUFFIX_NONE, SYNTAX_2OP\n\t"

__asm__ (intrinsic_2OP ("chk_pkt", 0x07, 0x01));

__extension__ static __inline int32_t __attribute__ ((__always_inline__))
__chk_pkt (int32_t __a)
{
    int32_t __dst;
    __asm__ ("chk_pkt %0, %1\n\t"
                : "=r" (__dst)
                : "rCal" (__a));
    return __dst;
}

int foo (void)
{
    return __chk_pkt (10);
}
```

Here is an assembler translation:

```gas
     .file   "t03.c"
     .cpu HS
     .extInstruction chk_pkt,0x07,0x01,SUFFIX_NONE, SYNTAX_2OP

     .section        .text
     .align 4
     .global foo
     .type   foo, @function
foo:
# 13 "t03.c" 1
     chk_pkt r0, 10

# 0 "" 2
     j_s [blink]
     .size   foo, .-foo
     .ident  "GCC: (ARCompact/ARCv2 ISA elf32 toolchain arc-2016.09-rc1-2-gb04a7b5) 6.2.1 20160824"
```

### Using Defines Only

Define a macro for a two-operand custom instruction in a header file:

```c
#define intrinsic_2OP(NAME, MOP, SOP)       \
    ".extInstruction " NAME "," #MOP ","    \
    #SOP ",SUFFIX_NONE, SYNTAX_2OP\n\t"
```

Create an inline function:

```c
__asm__ (intrinsic_2OP ("chk_pkt", 0x07, 0x01));
```

Define a macro for the custom instruction to be used in C sources:

```gas
#define chk_pkt(src) ({long __dst;                        \
       __asm__ ("chk_pkt %0, %1\n\t"                      \
              : "=r" (__dst)                              \
              : "rCal" (src));                            \
           __dst;})
```

Use the custom instruction in C-sources:

```c
result = chk_pkt(deltachk);
```

Here is a full content of the header file:

```c
#ifndef _EXT_INSTRUCTIONS_H_
#define _EXT_INSTRUCTIONS_H_

#define intrinsic_2OP(NAME, MOP, SOP)                                        \
    ".extInstruction " NAME "," #MOP "," #SOP ",SUFFIX_NONE, SYNTAX_2OP\n\t"

__asm__ (intrinsic_2OP ("chk_pkt", 0x07, 0x01));

#define chk_pkt(src) ({long __dst;                   \
        __asm__ ("chk_pkt %0, %1\n\t"                \
          : "=r" (__dst)                             \
          : "rCal" (src));                           \
         __dst;})

#endif /* _EXT_INSTRUCTIONS_H_ */
```
