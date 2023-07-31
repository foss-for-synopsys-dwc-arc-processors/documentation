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
