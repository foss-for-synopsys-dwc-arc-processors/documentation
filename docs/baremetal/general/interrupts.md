# Writing Interrupt Handlers

!!! info

    Refer [the official GCC documentation page](https://gcc.gnu.org/onlinedocs/gcc/ARC-Function-Attributes.html)
    about ARC function attributes for details.

## Overview

You can use `interrupt` GCC function attribute to declare a function as an
interrupt handler. Here is an example of declaring and defining a regular
interrupt handler for ARCv2 families:

```c
/* Declaration */
void f () __attribute__ ((interrupt ("ilink")));

/* Definition */
void f() {
    ...
}

/* Declaration with a definition */
__attribute__ ((interrupt ("ilink"))) void f() {
    ...
}
```

A parameter for `interrupt` attribute stands for a type of an interrupt:

| Parameter          | Type of an interrupt      |
|--------------------|---------------------------|
| `ilink`            | A regular ARCv2 interrupt |
| `firq`             | A fast ARCv2 interrupt    |
| `ilink1`, `ilink2` | ARC600/ARC700 interrupts  |

## Example

The `interrupt` attribute only puts valid entry and exit code around a function
for an interrupt or exception handler. Also, it's necessary to do extra steps:

* Provide an interrupt vector table (IVT) with an array of pointers to handlers.
  By default, it's provided only for ARCv2 targets in Newlib.
* Provide a valid value for `INT_VECTOR_BASE` auxiliary registers. It must
  contain a 1KB aligned address of IVT.

Here is a simple implementation of a handler for `EV_DivZero` exception for
ARCv2. It prints a message and then exits:

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define AUX_STATUS32_BCR 0x0A
#define AUX_STATUS32_DZ_OFFSET 13

/*
 * A helper for setting DZ flag - it enables EV_DivZero exceptions.
 */

void set_ev_divzero(bool on)
{
        unsigned int status32 = __builtin_arc_lr(AUX_STATUS32_BCR);
        unsigned int bit = (on ? 1 : 0) << AUX_STATUS32_DZ_OFFSET;
        status32 |= bit;
        __builtin_arc_flag(status32);
}

/*
 * Define an exception handler for EV_DivZero exception.
 * Note that it terminates the applications since EV_DivZero
 * returns to the same place in main function and the
 * exception is raised again.
 */

__attribute__ ((interrupt ("ilink"))) void EV_DivZero()
{
        printf("Divizion by zero is catched!\n");
        exit(1);
}

int main()
{
        /* Enable EV_DivZero exceptions */
        set_ev_divzero(true);

        /* Provoke EV_DivZero exception */
        int a = 5;
        int b = 0;
        int c = a / b;

        return 0;
}
```

Save it as `main.c` file and compile the application for ARC HS3x/4x families
using GNU toolchain:

```shell
arc-elf32-gcc -Wl,-marcv2elf -specs=hl.specs -mcpu=hs38 -O0 -g main.c -o main.elf
```

Meaning of the options:

* `-Wl,-marcv2elf` - [a linker emulation](./memory.md) which puts the IVT
  at `0x0`, this it's properly aligned.
* `-specs=hl.specs` - adds support of hostlink for nSIM simulator. It allows
  to use functions like `printf`.
* `-mcpu=hs38 -O0` - enable using `div` instructions and ensure that it's not
  reduces since our application is too simple.

Run the application using nSIM (`-p nsim_isa_intvbase_preset=0x0` is used to
ensure that `INT_VECTOR_BASE` is set to `0x0` by default):

```shell
$ nsimdrv -tcf $NSIM_HOME/etc/tcf/templates/hs48_full.tcf -p nsim_isa_intvbase_preset=0x0 main.elf
Divizion by zero is catched!
```
