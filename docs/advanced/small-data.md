# Small Data Sections

!!! warning

    These articles is applicable to ARC GCC starting only with 2017.09 release.

## Small Data Available Access Range

| Data type | Range          | Number of elements | Size       |
|-----------|----------------|--------------------|------------|
| `char`    | \[-256, 255\]  | 512                | 512 bytes  |
| `short`   | \[-256, 510\]  | 383                | 766 bytes  |
| `int`     | \[-256, 1020\] | 319                | 1276 bytes |

The lower limit depends on the possibility to access byte-aligned datum,
hence, it is hard connected to the range of `s9` short immediate (i.e.,
-256). Any other access can be done using address-scaling feature of the
load/store instructions.

The number of elements which we can fit in `sdata` section highly depends
on the data alignment properties. For example if we use only 4 byte
datum, 1 byte aligned, we can fit up to 128 elements in the section.

## Address Scaling Mode

To increase the access range for small data area, the compiler uses
scaled addresses whenever it is possible. Thus, we can extend
theoretically this range to \[-1024, 1020\] for 32-bit data
(e.g., `int`) if type aligned (i.e., 4-bytes). However, as we cannot be
100% sure that we address only 32-bit/4-byte aligned data, we need to
consider the worst case which is byte-aligned data. Thus, the effective
range is \[-256, 255\], with possibilities to access 16-bit aligned data
(e.g., `short`) up to 510, and 32-bit aligned data (e.g., `int` or
`long long`) up to 1020. While the lower limit remains -256. This
is because we set the linker script defined variable `__SDATA_BEGIN__`
with an offset of `0x100`. However, this rule can be overwritten by using
a custom linker script.

## Storing Global Data in Small Data Section

Global data smaller than a given number in bytes can be placed into the
`sdata` section. The number of bytes can be controlled via `-G<number>` option.
For ARC, by default this number is set to 8 whenever we have double
load/store operations available (i.e., ARC HS architecture), otherwise
to 4.

For example, a 8 bytes setting will allow us to place into `sdata` the
following variables:

```c
char gA[8];
short gB[4];
int gC[2];
long long gD;
```

Notable exceptions:

* Volatile global data will not be placed into `sdata` section when
  `-mno-volatile-cache` option is used.
* Strings and functions never end in small data area.
* Weak variables as well not.
* No constant will end in small data area as those one, we would like to place into ROM.

## Using Named Sections

Another way to control which data goes into small data area is to use
named sections like this:

```c
int a __attribute__((section (".sdata"))) = 1;
int b __attribute__((section (".sbss")));
```

Variables `a` and `b` will go into `sdata`/`sbss` sections without
checking the data type size against the `-G<number>` value. Thus, we
can always control which data is accessed via `gp` register by
setting `-G0` and using named sections.
