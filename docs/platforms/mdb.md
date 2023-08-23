# Using MetaWare Debugger

MetaWare Debugger may be used for debugging applications on various development
platforms via JTAG. Both Ashling Opella-XD probe and a simple USB cable for
connecting to Digilent chip may be used.

Commands listed below start a GUI version of MetaWare Debugger. Use `-cl`
option to run it in a command line mode.

Consider installing [Digilent Adept runtime and utilities](./digilent.md) if
you are going to debug development boards using MetaWare debugger.

## Debugging in a Single-core Mode

For Ashling Opella-XD:

```shell
mdb -DLL=opxdarc.so -prop=jtag_frequency=12MHz -prop=jtag_optimise=1 -nooptions -OK -memxfersize=0x8000 vmlinux
```

For Digilent:

```shell
mdb -digilent vmlinux
```

## Debugging in a Dual-core SMP Mode

For Ashling Opella-XD:

```shell
mdb -pset=1 -psetname=core0 -DLL=opxdarc.so -prop=jtag_frequency=12MHz -prop=jtag_optimise=1 -memxfersize=0x8000 vmlinux
mdb -pset=2 -psetname=core1 -DLL=opxdarc.so -prop=jtag_frequency=12MHz -prop=jtag_optimise=1 -prop=download=2 vmlinux
mdb -multifiles=core0,core1 -OK
```

For Digilent:

```shell
mdb -pset=1 -psetname=core0 -digilent vmlinux
mdb -pset=2 -psetname=core1 -digilent -prop=download=2 vmlinux
mdb -multifiles=core0,core1 -OK
```

## Debugging in a Quad-core SMP Mode

For Ashling Opella-XD:

```shell
mdb -pset=1 -psetname=core0 -DLL=opxdarc.so -prop=jtag_frequency=12MHz -prop=jtag_optimise=1 -memxfersize=0x8000 vmlinux
mdb -pset=2 -psetname=core1 -DLL=opxdarc.so -prop=jtag_frequency=12MHz -prop=jtag_optimise=1 -prop=download=2 vmlinux
mdb -pset=3 -psetname=core2 -DLL=opxdarc.so -prop=jtag_frequency=12MHz -prop=jtag_optimise=1 -prop=download=2 vmlinux
mdb -pset=4 -psetname=core3 -DLL=opxdarc.so -prop=jtag_frequency=12MHz -prop=jtag_optimise=1 -prop=download=2 vmlinux
mdb -multifiles=core0,core1,core2,core3 -OK
```

For Digilent:

```shell
mdb -pset=1 -psetname=core0 -digilent vmlinux
mdb -pset=2 -psetname=core1 -digilent -prop=download=2 vmlinux
mdb -pset=3 -psetname=core2 -digilent -prop=download=2 vmlinux
mdb -pset=4 -psetname=core3 -digilent -prop=download=2 vmlinux
mdb -multifiles=core0,core1,core2,core3 -OK
```

## Connecting to the Particular Device

You can choose a particular device connected to the host. It's possible when
it's connected through Digilent chip.

Firstly, obtain a name of the device you want to connect to using `djtgcfg`
utility from [Digilent Adept utilities](./digilent.md):

```text
$ djtgcfg enum
Found 1 device(s)

Device: HSDK-4xD
    Device Transport Type: 00020001 (USB)
    Product Name:          DesignWare ARC SDP
    User Name:             HSDK-4xD
    Serial Number:         251642000213
```

Set `DEVICE` variable and use it for debugging a quad-core HS Development Kit
target:

```shell
DEVICE=HSDK-4xD
mdb64 -pset=1 -cl -psetname=core0 -digilent -prop=dig_device=$DEVICE vmlinux
mdb64 -pset=2 -cl -psetname=core1 -digilent -prop=dig_device=$DEVICE -prop=download=2 vmlinux
mdb64 -pset=3 -cl -psetname=core2 -digilent -prop=dig_device=$DEVICE -prop=download=2 vmlinux
mdb64 -pset=4 -cl -psetname=core3 -digilent -prop=dig_device=$DEVICE -prop=download=2 vmlinux
mdb64 -cl -prop=dig_device=$DEVICE -multifiles=core0,core1,core2,core3 -OK
```

## Notes

* JTAG frequency should be at least 2 times lower than ARC core frequency.
  For 50MHz FPGA `-jtag_frequency=12MHz` is proven to work, while higher values
  may trigger problems like inconsistent memory values on elf load etc.
* `-memxfersize=0x8000` instructs debugger to send 32k blocks to Opella-XD.
  This significantly increases speed of elf download to target via JTAG.
* `-prop=jtag_optimise=1` turns off checking of the JTAG status registers by
  Opella-XD between successive JTAG read and write operations. This significantly
  increases speed of elf download to target via JTAG.
* `-prop=download=2` is required to let MDB know that this core will re-use
  elf loaded for other core
* To escape specification of full path to `opxdarc.so` create a symlink to the
  `.so` in `MW_HOME/MetaWare/arc/bin` like this:

    ```shell
    cd __path_to_opellaxdforarc_installation__
    ln -s $PWD/opxdarc.so $MW_HOME/MetaWare/arc/bin/opxdarc.so
    ```
